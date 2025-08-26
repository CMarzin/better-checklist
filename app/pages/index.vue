<script setup lang="ts">
import { h, resolveComponent } from 'vue'
import type { TableColumn } from '@nuxt/ui'
import type { Row } from '@tanstack/vue-table'
import type { Database } from '~/types/supabase'

const UButton = resolveComponent('UButton')
const UBadge = resolveComponent('UBadge')
const UDropdownMenu = resolveComponent('UDropdownMenu')

const supabase = useSupabaseClient<Database>()
const user = useSupabaseUser()

console.log(user)

const { data: versions, refresh, status } = await useAsyncData('versions', async () => {
	const { data } = await supabase.from('versions').select('*').order('id', { ascending: true })
	return data
})

const columns: TableColumn<Database['public']['Tables']['versions']['Row']>[] = ref([
	{
		accessorKey: 'id',
		header: '#',
		cell: ({ row }) => `#${row.getValue('id')}`
	},
	{
		accessorKey: 'number',
		header: 'Version Number',
	},
	{
		accessorKey: 'planned_date',
		header: 'Planned At',
		cell: ({ row }) => {
			return new Date(row.getValue('planned_date')).toLocaleString('en-US', {
				day: 'numeric',
				month: 'short',
				hour: '2-digit',
				minute: '2-digit',
				hour12: false
			})
		}
	},
	{
		accessorKey: 'status',
		header: 'Status',
		cell: ({ row }) => {
			const status = row.getValue('status')
			const color = () => {
				switch (status) {
					case 'PENDING':
						return 'info'
					case 'PUBLISHED':
						return 'success'
					case 'CANCELED':
						return 'error'
					default:
						return 'neutral'
				}
			}

			return h(UBadge, {
				size: 'md',
				variant: 'soft',
				color: color(),
				label: row.getValue('status')
			})
		}
	},
	{
		accessorKey: 'created_at',
		header: 'Created At',
		cell: ({ row }) => {
			return new Date(row.getValue('created_at')).toLocaleString('en-US', {
				day: 'numeric',
				month: 'short',
				hour: '2-digit',
				minute: '2-digit',
				hour12: false
			})
		}
	},
	{
		accessorKey: 'updated_at',
		header: 'Updated At',
		cell: ({ row }) => {
			return new Date(row.getValue('updated_at')).toLocaleString('en-US', {
				day: 'numeric',
				month: 'short',
				hour: '2-digit',
				minute: '2-digit',
				hour12: false
			})
		}
	},
	{
		id: 'actions',
		cell: ({ row }) => {
			return h(
				'div',
				{ class: 'text-right' },
				h(
					UDropdownMenu,
					{
						content: {
							align: 'end'
						},
						items: getRowItems(row),
						'aria-label': 'Actions dropdown'
					},
					() =>
						h(UButton, {
							icon: 'i-lucide-ellipsis-vertical',
							color: 'neutral',
							variant: 'ghost',
							class: 'ml-auto',
							'aria-label': 'Actions dropdown'
						})
				)
			)
		}
	}
])

function getRowItems(row: Row<Database['public']['Tables']['versions']['Row']>) {
	return [
		{
			type: 'label',
			label: 'Actions'
		},
		{
			label: 'Update version',
			to: `/version/${row.original.id || ''}/update`
		},
	]
}

</script>
<template>
	<div>
		<UButton @click="refresh">Refresh</UButton>
		<UTable :data="versions || []" :columns="columns" class="flex-1" :loading="status === 'pending'" />
	</div>
</template>
