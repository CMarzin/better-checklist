<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'
import type { Database } from '~/types/supabase'

const { id } = useRoute().params

const schema = z.object({
	versionsNumber: z.string()
		.min(1, 'Le numéro de version est requis')
		.regex(
			/^(?:0|[1-9]\d*)(?:\.(?:0|[1-9]\d*))?(?:\.(?:0|[1-9]\d*))?(?:-(?:(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+(?:[0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$/,
			'Le format du numéro de version est invalide. Exemples valides : 1.0.0, 2.1, 1.0.0-alpha, 3.2.1-beta.1'
		),
})

type Schema = z.output<typeof schema>

const state = reactive<Partial<Schema>>({
	versionsNumber: undefined,
})

const supabase = useSupabaseClient<Database>()

const toast = useToast()
async function onSubmit(event: FormSubmitEvent<Schema>) {
	const { error } = await supabase
		.from('versions')
		.update({ number: event.data.versionsNumber })
		.eq('id', Number(id))
		.select()

	if (error) {
		toast.add({ title: 'Error', description: error.message, color: 'error' })
	} else {
		toast.add({
			title: 'Success', description: 'The version has been updated.', color: 'success', actions: [{
				icon: 'i-lucide-home',
				label: 'Back to home',
				color: 'neutral',
				variant: 'outline',
				onClick: () => {
					navigateTo(`/`)
				}
			}]
		})
		
	}
}

const { data: version } = await useAsyncData('version', async () => {
	const { data } = await supabase.from('versions').select('*').eq('id', Number(id)).single()
	return data
})
</script>

<template>
	<div>
		<h1>Update Version</h1>
		<pre>{{ version }}</pre>
		<UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
			<UFormField label="Versions Number" name="versionsNumber">
				<UInput v-model="state.versionsNumber" />
			</UFormField>

			<UButton type="submit">
				Submit
			</UButton>
		</UForm>
	</div>
</template>
