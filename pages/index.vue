<template>
    <NuxtLayout>
        <div flex="~" w="screen" h="screen" items="center" text="white" justify="center" dir="rtl">
            <div w="md" items-center flex="~ col gap-6">
                <Logo />
                <!-- <span text="center 5xl white dark:white" mb="6">ElitesLiveSessions</span> -->
                <UiInput w="full" v-model="sessionCookie" label="معرف الجلسة" />
                <UiInput w="full" v-model="nameCookie" label="معرف الدخول" />
                <UiButton w="full" @click="join()" mt="4">دخول</UiButton>
                <span w="full" v-if="error" text="center sm errorOp dark:error" mt="4">{{ error }}</span>

                <div h="1px" w="full" bg="secondary dark:secondaryOp" my="4"></div>

                <UiButton w="full" to="/new-session" mt="4">انشاء جلسة جديدة</UiButton>
            </div>
        </div>
    </NuxtLayout>
</template>

<script setup>
const router = useRouter()
const nameCookie = useCookie('name')
const sessionCookie = useCookie('session')

const onlyEnglish = (str) => /^[A-Za-z0-9]*$/.test(str)

const error = ref(null)

const join = async () => {
    if (!onlyEnglish(nameCookie.value)) {
        error.value = "بالحروف الانجليزية وبدون مسافة"
        return
    }

    const sessionExists = await $fetch(`/api/session/${sessionCookie?.value}`)
    if (!sessionExists) {
        error.value = "الجلسة غير موجودة"
        return
    }

    router.push(`/sessions/${sessionCookie.value}`)
}

$fetch('/api/compile/')
</script>