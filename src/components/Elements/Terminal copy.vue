<template>
  <div
    class="w-full max-w-5xl bg-zinc-900 border-2 border-white/50 rounded-lg shadow-xl overflow-hidden"
  >
    <div class="bg-black p-2 flex items-center space-x-2">
      <div class="w-3 h-3 rounded-full bg-red-500"></div>
      <div class="w-3 h-3 rounded-full bg-yellow-500"></div>
      <div class="w-3 h-3 rounded-full bg-green-500"></div>
      <div class="text-gray-400 text-sm font-mono ml-2">tanjun@workflow:~$</div>
    </div>
    <div
      class="p-4 h-[33rem] overflow-y-scroll font-mono text-lg text-gray-300 terminal-content"
      ref="terminalOutput"
    >
      <div v-for="(line, index) in outputLines" :key="index">
        <template v-if="line.type === 'command'">
          <span class="text-green-400">tanjun@workflow:~$</span>
          {{ line.content }}
        </template>
        <template v-else-if="line.type === 'prompt'">
          <div class="mb-5">
            <span class="text-blue-400">{{ line.label }}</span
            ><br />
            <span class="text-gray-500">{{ line.description }}</span
            ><br />
            <span class="text-white"
              ><span class="text-pink-400">></span> {{ line.value }}</span
            >
          </div>
        </template>
        <template v-else-if="line.type === 'progress'">
          <div>
            {{ line.content }} ({{ line.progress }}%) [{{
              '#'.repeat(Math.floor(line.progress / 2)).padEnd(50, '.')
            }}]
          </div>
        </template>
        <template v-else>
          <span class="whitespace-pre-wrap">{{ line.content }}</span>
        </template>
      </div>
      <div class="flex items-center">
        <span class="text-green-400">tanjun@workflow:~$</span>
        <span class="ml-2">{{ currentCommand }}</span>
        <span class="animate-blink ml-1">▋</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, watch, reactive } from 'vue'

const outputLines = ref([])
const currentCommand = ref('')
const terminalOutput = ref(null)

const workflow = [
  {
    command: 'tanjun init',
    output: [
      {
        type: 'prompt',
        label: 'Project Name',
        description: 'The name of the project',
        value: 'bun',
        duration: 1000,
      },
      {
        type: 'prompt',
        label: 'Image Name',
        description: 'The image name should be in the format of registry/image',
        value: 'ghcr.io/shyim',
        duration: 1500,
      },
      {
        type: 'prompt',
        label: 'SSH Address',
        description: 'The SSH address of the server',
        value: 'example.com',
        duration: 1000,
      },
      {
        type: 'prompt',
        label: 'SSH User',
        description: 'The SSH user to connect to the server',
        value: 'root',
        duration: 1000,
      },
      {
        type: 'prompt',
        label: 'SSH Port',
        description: 'The SSH port to connect to the server',
        value: '22',
        duration: 1000,
      },
      {
        type: 'prompt',
        label: 'Host',
        description: 'The host to use for the server',
        value: 'myapp.example.com',
        duration: 1000,
      },
      {
        type: 'prompt',
        label: 'Use HTTPS',
        description: 'Use HTTPS for the server',
        value: 'Yes',
        duration: 1000,
      },
      {
        type: 'output',
        content:
          'Created a .tanjun.yml. Run next tanjun setup to setup the server',
        duration: 1000,
      },
    ],
  },
  {
    command: 'tanjun setup',
    output: [
      {
        type: 'output',
        content: 'Setting up Proxy Server on the remote server...',
        duration: 1000,
      },
      {
        type: 'progress',
        content: 'Connecting to remote server',
        duration: 3000,
      },
      { type: 'progress', content: 'Installing dependencies', duration: 5000 },
      { type: 'progress', content: 'Configuring Proxy Server', duration: 4000 },
      {
        type: 'output',
        content: 'Proxy Server setup complete!',
        duration: 1000,
      },
    ],
  },
  // ... (other workflow steps remain unchanged)
]

const simulateTyping = async (text, minDelay = 20, maxDelay = 100) => {
  let result = ''
  for (let i = 0; i < text.length; i++) {
    result += text[i]
    currentCommand.value = result
    await nextTick()
    const delay = Math.random() * (maxDelay - minDelay) + minDelay
    await new Promise(resolve => setTimeout(resolve, delay))
  }
  return result
}

const simulateProgress = async (line, duration) => {
  const startTime = Date.now()
  const endTime = startTime + duration

  return new Promise(resolve => {
    function updateProgress() {
      const now = Date.now()
      const progress = Math.min((now - startTime) / duration, 1)
      line.progress = Math.floor(progress * 100)

      if (now < endTime) {
        requestAnimationFrame(updateProgress)
      } else {
        resolve()
      }
    }

    requestAnimationFrame(updateProgress)
  })
}

const executeWorkflow = async () => {
  for (const step of workflow) {
    currentCommand.value = ''
    await simulateTyping(step.command)
    await new Promise(resolve => setTimeout(resolve, 500))

    outputLines.value.push({ type: 'command', content: step.command })
    currentCommand.value = ''

    for (const item of step.output) {
      if (item.type === 'prompt') {
        const promptLine = reactive({ ...item, value: '' })
        outputLines.value.push(promptLine)
        await simulateTyping(item.value, 50, 150)
        await new Promise(resolve => setTimeout(resolve, item.duration))
        currentCommand.value = ''
        promptLine.value = item.value
      } else if (item.type === 'progress') {
        const progressLine = reactive({ ...item, progress: 0 })
        outputLines.value.push(progressLine)
        await simulateProgress(progressLine, item.duration)
      } else {
        outputLines.value.push(item)
        await new Promise(resolve => setTimeout(resolve, item.duration))
      }
      await nextTick()
    }

    await new Promise(resolve => setTimeout(resolve, 1000))
  }

  currentCommand.value = ''
}

// Auto-scroll function
const scrollToBottom = () => {
  if (terminalOutput.value) {
    terminalOutput.value.scrollTop = terminalOutput.value.scrollHeight
  }
}

// Watch for changes in outputLines and currentCommand
watch([outputLines.value, currentCommand], () => {
  nextTick(() => {
    scrollToBottom()
  })
})

onMounted(async () => {
  await executeWorkflow()
})
</script>

<style scoped>
@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

.animate-blink {
  animation: blink 1s step-end infinite;
}

.terminal-content::-webkit-scrollbar {
  background-color: transparent;
  width: 0.3125rem;
}

.terminal-content::-webkit-scrollbar-thumb {
  background-color: #babac08c;
  border-radius: 1rem;
  transition: background-color ease 0.2s;
}

::-webkit-scrollbar-thumb:hover {
  background-color: #dddde6b4;
}
</style>
