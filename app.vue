<script setup lang="ts">
const WHISPER_SAMPLING_RATE = 16_000
const MAX_AUDIO_LENGTH = 30 // seconds
const MAX_SAMPLES = WHISPER_SAMPLING_RATE * MAX_AUDIO_LENGTH

type WorkerMessage =
    | { status: 'loading'; data: string }
    | { status: 'initiate'; file: string; progress: number; total: number }
    | { status: 'progress'; file: string; progress: number; total: number }
    | { status: 'done'; file: string }
    | { status: 'ready' }
    | { status: 'start' }
    | { status: 'update'; tps: number }
    | { status: 'complete'; output: string }

interface ProgressItem {
    file: string
    progress: number
    total: number
}

const worker = ref<Worker>()
const recorder = ref<MediaRecorder | null>(null)
const status = ref<string | null>(null)
const loadingMessage = ref('')
const progressItems = ref<ProgressItem[]>([])
const text = ref('')
const tps = ref<number | null>(null)
const language = 'en'
const recording = ref(false)
const isProcessing = ref(false)
const chunks = ref<Blob[]>([])
const stream = ref<MediaStream | null>(null)
const audioContext = ref<AudioContext | null>(null)

const onMessageReceived = (e: MessageEvent<WorkerMessage>) => {
    switch (e.data.status) {
        case 'loading':
            // Model file start load: add a new progress item to the list.
            status.value = 'loading'
            loadingMessage.value = e.data.data
            break

        case 'initiate':
            progressItems.value = [...progressItems.value, e.data]
            break

        case 'progress':
            // Model file progress: update one of the progress items.
            progressItems.value = progressItems.value.map(item => {
                if (item.file === e.data.file) {
                    return { ...item, ...e.data }
                }
                return item
            })

            break

        case 'done':
            progressItems.value = progressItems.value.map(prev =>
                prev.filter(item => item.file !== e.data.file)
            )
            break

        case 'ready':
            status.value = 'ready'
            recorder.value?.start()
            break

        case 'start':
            isProcessing.value = true
            recorder.value?.requestData()
            break

        case 'update':
            const { tps: _tps } = e.data
            tps.value = _tps
            break

        case 'complete':
            isProcessing.value = false
            text.value = e.data.output
            break
    }
}

onMounted(() => {
    if (!worker.value) {
        // Create the worker if it does not yet exist.
        worker.value = new Worker(new URL('./lib/worker.ts', import.meta.url), {
            type: 'module',
        })
    }

    worker.value.addEventListener('message', onMessageReceived)
})

onMounted(() => {
    if (recorder.value) return // Already set

    if (navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices
            .getUserMedia({ audio: true })
            .then(s => {
                stream.value = s

                recorder.value = new MediaRecorder(s)
                audioContext.value = new AudioContext({
                    sampleRate: WHISPER_SAMPLING_RATE,
                })

                recorder.value.onstart = () => {
                    recording.value = true
                    chunks.value = []
                }
                recorder.value.ondataavailable = e => {
                    if (e.data.size > 0) {
                        chunks.value = [...chunks.value, e.data]
                    } else {
                        // Empty chunk received, so we request new data after a short timeout
                        setTimeout(() => {
                            recorder.value?.requestData()
                        }, 25)
                    }
                }

                recorder.value.onstop = () => {
                    recording.value = false
                }
            })
            .catch(err => console.error('The following error occurred: ', err))
    } else {
        console.error('getUserMedia not supported on your browser!')
    }
})

watch([status, recording, isProcessing, chunks], () => {
    if (!recorder.value) return
    if (!recording.value) return
    if (isProcessing.value) return
    if (status.value !== 'ready') return

    if (chunks.value.length > 0) {
        // Generate from data
        const blob = new Blob(chunks.value, { type: recorder.value.mimeType })

        const fileReader = new FileReader()

        fileReader.onloadend = async () => {
            const arrayBuffer = fileReader.result
            const decoded = await audioContext.value?.decodeAudioData(
                arrayBuffer as ArrayBuffer
            )
            let audio = decoded?.getChannelData(0)
            if (audio && audio.length > MAX_SAMPLES) {
                // Get last MAX_SAMPLES
                audio = audio.slice(-MAX_SAMPLES)
            }

            worker.value?.postMessage({
                type: 'generate',
                data: { audio, language },
            })
        }
        fileReader.readAsArrayBuffer(blob)
    } else {
        recorder.value?.requestData()
    }
})

onUnmounted(() => {
    worker.value?.removeEventListener('message', onMessageReceived)
})
</script>

<template>
    <div
        class="flex flex-col h-screen mx-auto justify-end text-gray-800 dark:text-gray-200 bg-white dark:bg-gray-900"
    >
        <div
            class="h-full overflow-auto scrollbar-thin flex justify-center items-center flex-col relative"
        >
            <div
                class="flex flex-col items-center mb-1 max-w-[400px] text-center"
            >
                <h1 class="text-4xl font-bold mb-1">Whisper WebGPU</h1>
                <h2 class="text-xl font-semibold">
                    Real-time in-browser speech recognition
                </h2>
            </div>

            <div class="flex flex-col items-center px-4">
                <template v-if="status === null">
                    <button
                        class="border px-4 py-2 rounded-lg bg-blue-400 text-white hover:bg-blue-500 disabled:bg-blue-100 disabled:cursor-not-allowed select-none"
                        @click="
                            () => {
                                worker?.postMessage({ type: 'load' })
                                status = 'loading'
                            }
                        "
                        :disabled="status !== null"
                    >
                        Load model
                    </button>
                </template>

                <div class="w-[500px] p-2">
                    <AudioVisualizer
                        class="w-full rounded-lg"
                        :stream="stream"
                    />
                    <div v-if="status === 'ready'" class="relative">
                        <p
                            class="w-full h-[80px] overflow-y-auto overflow-wrap-anywhere border rounded-lg p-2"
                        >
                            {{ text }}
                        </p>
                        <span v-if="tps" class="absolute bottom-0 right-0 px-1">
                            {{ tps.toFixed(2) }} tok/s
                        </span>
                    </div>
                </div>

                <div
                    v-if="status === 'loading'"
                    class="w-full max-w-[500px] text-left mx-auto p-4"
                >
                    <p class="text-center">{{ loadingMessage }}</p>
                    <Progress
                        v-for="(item, i) in progressItems"
                        :key="i"
                        :text="item.file"
                        :percentage="item.progress"
                        :total="item.total"
                    />
                </div>
            </div>
        </div>
    </div>
</template>
