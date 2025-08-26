<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'
import type { Database } from '~/types/supabase'

const schema = z.object({
	versionsNumber: z.string().min(1, 'Must be at least 1 character'),
})

type Schema = z.output<typeof schema>

const state = reactive<Partial<Schema>>({
	versionsNumber: undefined,
})

const supabase = useSupabaseClient<Database>()

const toast = useToast()
async function onSubmit(event: FormSubmitEvent<Schema>) {
	const { data, error } = await supabase
		.from('versions')
		.insert([
			{ number: event.data.versionsNumber },
		])
		.select()
	if (error) {
		toast.add({ title: 'Error', description: error.message, color: 'error' })
	} else {
		toast.add({ title: 'Success', description: 'The form has been submitted.', color: 'success' })
	}
	console.log(data)
}
</script>

<template>
	<UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
		<UFormField label="Versions Number" name="versionsNumber">
			<UInput v-model="state.versionsNumber" />
		</UFormField>

		<UButton type="submit">
			Submit
		</UButton>
	</UForm>
</template>
