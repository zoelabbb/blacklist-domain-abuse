<script lang="ts">
	export let data: Array<{
		jenisLaporan: string;
		url: string;
		status: string;
		target: string;
		reporter: string;
		tanggalLaporan: string;
	}> = [];
</script>

<div class="overflow-x-auto">
	<table class="min-w-full divide-y divide-gray-200">
		<thead class="bg-gray-50">
			<tr>
				<th
					scope="col"
					class="px-3 py-2 text-left text-xs font-medium tracking-wider text-gray-500 uppercase sm:px-6 sm:py-3"
				>
					Jenis
				</th>
				<th
					scope="col"
					class="px-3 py-2 text-left text-xs font-medium tracking-wider text-gray-500 uppercase sm:px-6 sm:py-3"
				>
					URL
				</th>
				<th
					scope="col"
					class="hidden px-3 py-2 text-left text-xs font-medium tracking-wider text-gray-500 uppercase sm:table-cell sm:px-6 sm:py-3"
				>
					Status
				</th>
				<th
					scope="col"
					class="hidden px-3 py-2 text-left text-xs font-medium tracking-wider text-gray-500 uppercase sm:table-cell sm:px-6 sm:py-3"
				>
					Target
				</th>
				<th
					scope="col"
					class="hidden px-3 py-2 text-left text-xs font-medium tracking-wider text-gray-500 uppercase sm:table-cell sm:px-6 sm:py-3"
				>
					Reporter
				</th>
				<th
					scope="col"
					class="hidden px-3 py-2 text-left text-xs font-medium tracking-wider text-gray-500 uppercase sm:table-cell sm:px-6 sm:py-3"
				>
					Tanggal
				</th>
			</tr>
		</thead>
		<tbody class="divide-y divide-gray-200 bg-white">
			{#each data as item (item.url)}
				<tr class="hover:bg-gray-50">
					<td class="px-3 py-3 whitespace-nowrap sm:px-6 sm:py-4">
						<span
							class="inline-flex rounded-full px-2 py-1 text-xs leading-4 font-semibold
                {item.jenisLaporan === 'Phishing'
								? 'bg-red-100 text-red-800'
								: item.jenisLaporan === 'Malware'
									? 'bg-purple-100 text-purple-800'
									: 'bg-gray-100 text-gray-800'}"
						>
							{item.jenisLaporan.slice(0, 3)}
							<span class="hidden sm:inline">.{item.jenisLaporan.slice(3)}</span>
						</span>
					</td>
					<td
						class="max-w-[120px] truncate px-3 py-3 whitespace-nowrap sm:max-w-none sm:px-6 sm:py-4"
					>
						<a
							href={item.url}
							target="_blank"
							rel="noopener noreferrer"
							class="text-xs text-blue-600 hover:underline sm:text-sm"
						>
							{item.url.replace(/^https?:\/\//, '')}
						</a>
					</td>
					<td class="hidden px-3 py-3 whitespace-nowrap sm:table-cell sm:px-6 sm:py-4">
						<span
							class="inline-flex rounded-full px-2 py-1 text-xs leading-4 font-semibold
                {item.status === 'Aktif'
								? 'bg-green-100 text-green-800'
								: 'bg-gray-100 text-gray-800'}"
						>
							{item.status}
						</span>
					</td>
					<td
						class="hidden px-3 py-3 text-xs whitespace-nowrap text-gray-500 sm:table-cell sm:px-6 sm:py-4 sm:text-sm"
					>
						{item.target}
					</td>
					<td
						class="hidden px-3 py-3 text-xs whitespace-nowrap text-gray-500 sm:table-cell sm:px-6 sm:py-4 sm:text-sm"
					>
						{item.reporter.split(' ')[0]}
						<span class="hidden sm:inline"> {item.reporter.split(' ').slice(1).join(' ')}</span>
					</td>
					<td
						class="hidden px-3 py-3 text-xs whitespace-nowrap text-gray-500 sm:table-cell sm:px-6 sm:py-4 sm:text-sm"
					>
						{item.tanggalLaporan.split('-').reverse().join('/')}
					</td>
				</tr>
			{/each}
		</tbody>
	</table>
</div>
