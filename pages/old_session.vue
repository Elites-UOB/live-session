<template>
    <NuxtLayout>
        <div h="full" flex="~">
            <!-- left -->
            <div flex="~ col basis-1/2">
                <!-- <iframe border="0" id="preview" flex="grow" w="full" ref="previewRef"></iframe> -->
                <span>Is Admin: {{ isAdmin }}</span>
                <span>
                    Read Only: {{ sessionData?.readOnly }} <UiButton @click="toggleSessionState" w="fit">Toggle State</UiButton>
                </span>
                <p>
                    {{code.html}}
                </p>
            </div>


            <!-- right -->
            <div flex="~ col gap-4 basis-1/2" border="l-2 secondary dark:secondaryOp">
                <!-- Tabs -->
                <UiTabGroup flex="basis-1/2" :tabs="['html', 'css', 'js']">
                    <template #tab-1>
                        <MonacoEditor w="full" h="full" @keyup="sendCode($event)" v-model="code.html" lang="html" :options="{ theme: 'vs-dark', fontSize: '20px', readOnly: !isAdmin && sessionData?.readOnly }" />
                    </template>

                    <template #tab-2>
                        <MonacoEditor w="full" h="full" @keyup="sendCode($event)" v-model="code.css" lang="css" :options="{ theme: 'vs-dark', fontSize: '20px', readOnly: !isAdmin && sessionData?.readOnly }" />
                    </template>

                    <template #tab-3>
                        <MonacoEditor w="full" h="full" @keyup="sendCode($event)" v-model="code.javascript" lang="javascript" :options="{ theme: 'vs-dark', fontSize: '20px', readOnly: !isAdmin && sessionData?.readOnly }" />
                    </template>
                </UiTabGroup>
                <div flex="~ basis-1/2">
                    <iframe h="full" w="full" ref="previewRef" border="0"></iframe>
                </div>
            </div>
        </div>
    </NuxtLayout>

</template>

<script setup>
const router = useRouter()
// Session ID & Person Name
const { params } = useRoute()
const { session } = params
const name = useCookie('name')
const userId = useCookie('userId')
// IFrame Reference
const previewRef = ref(null)
// Current Selected Language
const selectedLanguage = ref('html')

const socket = useSocket()

const sessionData = ref(null)
const isAdmin = computed(() => sessionData.value?.adminId === userId?.value)

// Code with initated values
const code = reactive({
    html: '',
    css: '',
    javascript: ''
})

// Languages
const languages = [
    { name: 'HTML', value: 'html' },
    { name: 'CSS', value: 'css' },
    { name: 'JS', value: 'javascript' },
]

// Update preview
const updatePreview = useDebounceFn(async () => {
    const { html, css, javascript } = code

    // Get element of iframe
    const preview = previewRef.value.contentDocument || previewRef.value.contentWindow.document;

    if (!preview) return false

    preview.open();
    preview.write(`<style scoped>${css ?? ''}</style>${html ?? ''}`);

    const scriptEl = document.createElement('script');
    const newContent = document.createRange().createContextualFragment(javascript ?? '');
    scriptEl.append(newContent)
    preview.body.appendChild(scriptEl);
    preview.close();

}, 250)


const sendCode = useDebounceFn(async (e) => {
    const prevents = [16, 17, 18, 19, 20, 27, 33, 34, 35, 36, 37, 38, 39, 40, 44, 45, 91, 92, 93, 112, 113, 114, 115, , 116, 117, 118, 119, 120, 121, 122, 123, 144, 145, 173, 174, 175, 181, 182, 183]
    if (prevents.includes(e.which)) {
        console.log('not a letter')
        return false
    }

    if (socket.value.data) {
        const { type, data: receivedData } = JSON.parse(socket.value.data)
        if (code.html == receivedData.html && code.css == receivedData.css && code.javascript == receivedData.javascript) return false
    }

    await $fetch('/api/code', {
        method: 'POST',
        body: JSON.stringify({
            message: code,
        })
    })
}, 250)


watch(() => JSON.parse(socket.value.data), async (data) => {
    const { type, data: receivedData } = data
    if (type == "code") {
        code.html = receivedData.html
        code.css = receivedData.css
        code.javascript = receivedData.javascript

    } else if (type == "refresh") {
        sessionData.value = await $fetch(`/api/session/${session}`, {
            headers: {
                'Cache-Control': 'no-cache'
            }
        })
    } else {
        console.log(data.value)
    }
}, { deep: true })

// Update preview on mounted
// Send to the web socket server 
// $socket.send("test")
try {
    sessionData.value = await $fetch(`/api/session/${session}`)
} catch (e) {
    router.push('/')
}

onMounted(() => {
    if (sessionData.value?.data.type == "code") {
        code.html = sessionData.value?.data.data.html
        code.css = sessionData.value?.data.data.css
        code.javascript = sessionData.value?.data.data.javascript
        updatePreview()
    }
})



// Listen for writes in editors
watchDebounced(() => code, () => updatePreview(), { deep: true, debounce: 250, maxWait: 1000 })




const toggleSessionState = async () => {
    await $fetch(`/api/session/${session}/toggleState`, {
        method: 'POST',
        headers: {
            'Cache-Control': 'no-cache'
        }
    })
}

</script>