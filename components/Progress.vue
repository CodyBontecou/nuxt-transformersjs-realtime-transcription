<script setup lang="ts">
defineProps<{
    text?: string
    percentage?: number
    total?: number
}>()

const formatBytes = (size: number) => {
    const i = size === 0 ? 0 : Math.floor(Math.log(size) / Math.log(1024))
    return `${(size / Math.pow(1024, i)).toFixed(2)} ${
        ['B', 'kB', 'MB', 'GB', 'TB'][i]
    }`
}
</script>

<template>
    <div
        class="w-full bg-gray-100 dark:bg-gray-700 text-left rounded-lg overflow-hidden mb-0.5"
    >
        <div
            class="bg-blue-400 whitespace-nowrap px-1 text-sm"
            :style="{ width: `${percentage ?? 0}%` }"
        >
            {{ text }} ({{ (percentage ?? 0).toFixed(2) }}%
            <span v-if="!isNaN(total ?? NaN)">
                of {{ formatBytes(total!) }}</span
            >)
        </div>
    </div>
</template>
