<script lang="ts">
	import { onMount } from 'svelte';

	const API_URL = import.meta.env.VITE_API_URL;
	const API_KEY = import.meta.env.VITE_API_KEY;

	interface BlacklistItem {
		id: string;
		url: string;
		status: string;
		target: string;
		reporter: string;
		created_at: string;
	}

	// Data state
	let blacklistData: BlacklistItem[] = [];
	let filteredData: BlacklistItem[] = [];
	let displayedData: BlacklistItem[] = [];
	let isLoading = true;
	let error: string | null = null;

	// Filter state
	let searchQuery = '';
	let statusFilter = 'Semua Status';
	let typeFilter = 'Semua Jenis';
	let sortOption = 'Terbaru';

	// Pagination state
	let currentPage = 1;
	const itemsPerPage = 6;
	let totalPages = 1;

	async function fetchBlacklist() {
		try {
			const res = await fetch(`${API_URL}/api/v1/data-phishing`, {
				method: 'GET',
				headers: {
					'X-API-KEY': API_KEY,
					'Content-Type': 'application/json'
				}
			});

			if (!res.ok) {
				throw new Error('Gagal mengambil data blacklist');
			}

			const response = await res.json();
			blacklistData = response.data.map((item: BlacklistItem) => ({
				...item,
				tanggalLaporan: item.created_at ? item.created_at.split('T')[0] : 'Tidak ada tanggal'
			}));
			filterData();
		} catch (err) {
			error = (err as Error).message;
			console.error('Error fetching data:', err);
		} finally {
			isLoading = false;
		}
	}

	function filterData() {
		filteredData = [...blacklistData];

		// Apply search filter
		if (searchQuery) {
			const query = searchQuery.toLowerCase();
			filteredData = filteredData.filter(
				(item) =>
					item.url.toLowerCase().includes(query) ||
					item.target.toLowerCase().includes(query) ||
					item.reporter.toLowerCase().includes(query) ||
					item.status.toLowerCase().includes(query)
			);
		}

		// Apply status filter
		if (statusFilter !== 'Semua Status') {
			filteredData = filteredData.filter((item) => item.status === statusFilter);
		}

		// Apply type filter
		if (typeFilter !== 'Semua Jenis') {
			filteredData = filteredData.filter((item) => item.jenisLaporan === typeFilter);
		}

		// Apply sorting
		if (sortOption === 'Terbaru') {
			filteredData.sort(
				(a, b) => new Date(b.created_at).getTime() - new Date(a.created_at).getTime()
			);
		} else if (sortOption === 'Terlama') {
			filteredData.sort(
				(a, b) => new Date(a.created_at).getTime() - new Date(b.created_at).getTime()
			);
		}

		// Reset to first page when filters change
		currentPage = 1;
		updatePagination();
	}

	function updatePagination() {
		// Calculate total pages
		totalPages = Math.ceil(filteredData.length / itemsPerPage);

		// Get data for current page
		const startIndex = (currentPage - 1) * itemsPerPage;
		displayedData = filteredData.slice(startIndex, startIndex + itemsPerPage);
	}

	function goToPage(page: number) {
		if (page >= 1 && page <= totalPages) {
			currentPage = page;
			updatePagination();
		}
	}

	// Event handlers
	function handleSearch(e: Event) {
		searchQuery = (e.target as HTMLInputElement).value;
		filterData();
	}

	function handleStatusFilter(e: Event) {
		statusFilter = (e.target as HTMLSelectElement).value;
		filterData();
	}

	function handleTypeFilter(e: Event) {
		typeFilter = (e.target as HTMLSelectElement).value;
		filterData();
	}

	function handleSort(e: Event) {
		sortOption = (e.target as HTMLSelectElement).value;
		filterData();
	}

	onMount(async () => {
		await fetchBlacklist();
	});
</script>

<!-- SEO -->
<svelte:head>
	<title>Daftar Blacklist</title>
	<meta name="description" content="Daftar URL yang dilaporkan berbahaya" />
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
	<meta name="author" content="Raksaka Security" />
	<meta name="keywords" content="blacklist, laporan, keamanan, phishing, malware" />
</svelte:head>

<div class="min-h-screen bg-gray-50 p-4 sm:p-6">
	<main class="mx-auto max-w-7xl">
		<!-- Header Section -->
		<div class="mb-6 flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
			<div>
				<h1 class="text-2xl font-bold text-gray-900 sm:text-3xl">Daftar Blacklist</h1>
				<p class="mt-1 text-sm text-gray-600 sm:text-base">Daftar URL yang dilaporkan berbahaya</p>
			</div>
			<div class="flex flex-wrap gap-2 sm:space-x-3">
				<a
					href="https://raksaka.urproject.my.id/report"
					class="flex items-center rounded-lg bg-blue-600 px-3 py-1.5 text-sm text-white transition hover:bg-blue-700 sm:px-4 sm:py-2 sm:text-base"
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="mr-1 h-4 w-4 sm:mr-2 sm:h-5 sm:w-5"
						viewBox="0 0 20 20"
						fill="currentColor"
					>
						<path
							fill-rule="evenodd"
							d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z"
							clip-rule="evenodd"
						/>
					</svg>
					<span class="sr-only sm:not-sr-only">Report</span>
				</a>
			</div>
		</div>

		<!-- Filter and Search Section -->
		<div class="mb-4 rounded-lg bg-white p-3 shadow-sm sm:mb-6 sm:p-4">
			<div class="flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
				<!-- Search Input -->
				<div class="relative md:w-64 lg:w-80">
					<div class="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3">
						<svg
							class="h-4 w-4 text-gray-400 sm:h-5 sm:w-5"
							fill="currentColor"
							viewBox="0 0 20 20"
						>
							<path
								fill-rule="evenodd"
								d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"
								clip-rule="evenodd"
							></path>
						</svg>
					</div>
					<input
						type="text"
						class="block w-full rounded-lg border border-gray-300 py-1.5 pr-3 pl-9 text-sm focus:border-blue-500 focus:ring-blue-500 focus:outline-none sm:py-2 sm:pl-10 sm:text-base"
						placeholder="Cari..."
						bind:value={searchQuery}
						on:input={handleSearch}
					/>
				</div>

				<!-- Filters -->
				<div class="grid w-full grid-cols-2 gap-2 sm:flex sm:flex-nowrap sm:gap-3 md:w-auto">
					<select
						class="w-full rounded-lg border border-gray-300 px-2 py-1.5 text-xs focus:border-blue-500 focus:ring-blue-500 focus:outline-none sm:px-3 sm:py-2 sm:text-sm"
						bind:value={typeFilter}
						on:change={handleTypeFilter}
					>
						<option>Semua Jenis</option>
						<option>Phishing</option>
						<option>Malware</option>
					</select>
					<select
						class="w-full rounded-lg border border-gray-300 px-2 py-1.5 text-xs focus:border-blue-500 focus:ring-blue-500 focus:outline-none sm:px-3 sm:py-2 sm:text-sm"
						bind:value={statusFilter}
						on:change={handleStatusFilter}
					>
						<option>Semua Status</option>
						<option>active</option>
						<option>pending</option>
						<option>inactive</option>
					</select>
					<select
						class="col-span-2 w-full rounded-lg border border-gray-300 px-2 py-1.5 text-xs focus:border-blue-500 focus:ring-blue-500 focus:outline-none sm:col-span-1 sm:px-3 sm:py-2 sm:text-sm"
						bind:value={sortOption}
						on:change={handleSort}
					>
						<option>Urutkan</option>
						<option>Terbaru</option>
						<option>Terlama</option>
					</select>
				</div>
			</div>
		</div>

		<!-- Table Section -->
		<div class="overflow-hidden rounded-lg bg-white shadow-sm">
			{#if isLoading}
				<div class="flex justify-center p-8">
					<div
						class="h-8 w-8 animate-spin rounded-full border-4 border-blue-500 border-t-transparent"
					></div>
				</div>
			{:else if error}
				<div class="p-4 text-center text-red-600">{error}</div>
			{:else if filteredData.length === 0}
				<div class="p-4 text-center text-gray-500">
					{#if searchQuery || statusFilter !== 'All Statuses' || typeFilter !== 'All Types'}
						No data matches your filters.
					{:else}
						No data found.
					{/if}
				</div>
			{:else}
				<!-- Mobile View (Cards) -->
				<div class="sm:hidden">
					{#each filteredData as item}
						<div class="border-b border-gray-200 p-4 last:border-b-0">
							<div class="flex items-start justify-between">
								<div class="truncate font-medium text-gray-900">{item.url}</div>
								<span
									class="ml-2 inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium
									{item.status === 'approved'
										? 'bg-green-100 text-green-800'
										: item.status === 'pending'
											? 'bg-yellow-100 text-yellow-800'
											: 'bg-red-100 text-red-800'}"
								>
									{item.status}
								</span>
							</div>
							<div class="mt-2 text-sm text-gray-500">
								<div class="flex items-center">
									<svg class="mr-1.5 h-4 w-4 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
										<path
											fill-rule="evenodd"
											d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z"
											clip-rule="evenodd"
										></path>
									</svg>
									{item.reporter}
								</div>
								<div class="flex items-center">
									<svg class="mr-1.5 h-4 w-4 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
										<path
											fill-rule="evenodd"
											d="M6 2a1 1 0 00-1 1v1H4a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-1V3a1 1 0 10-2 0v1H7V3a1 1 0 00-1-1zm0 5a1 1 0 000 2h8a1 1 0 100-2H6z"
											clip-rule="evenodd"
										></path>
									</svg>
									{item.created_at
										? new Date(item.created_at).toLocaleDateString('id-ID', {
												year: 'numeric',
												month: '2-digit',
												day: '2-digit'
											})
										: 'Tidak ada tanggal'}
								</div>
								<div class="flex items-center">
									<svg class="mr-1.5 h-4 w-4 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
										<path
											d="M10 2a6 6 0 00-6 6v3.586l-.707.707A1 1 0 004 14h12a1 1 0 00.707-1.707L16 11.586V8a6 6 0 00-6-6zM10 18a3 3 0 01-3-3h6a3 3 0 01-3 3z"
										></path>
									</svg>
									{item.target}
								</div>
							</div>
							<div class="mt-2 flex justify-end">
								<a
									href={`/detail/${item.id}`}
									class="text-sm font-medium text-blue-600 hover:text-blue-500">Detail</a
								>
							</div>
						</div>
					{/each}
				</div>

				<!-- Desktop View (Table) -->
				<div class="hidden sm:block">
					<table class="min-w-full divide-y divide-gray-200">
						<thead class="bg-gray-50">
							<tr>
								<th
									scope="col"
									class="px-6 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>URL</th
								>
								<th
									scope="col"
									class="px-6 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Status</th
								>
								<th
									scope="col"
									class="px-6 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Target</th
								>
								<th
									scope="col"
									class="px-6 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Pelapor</th
								>
								<th
									scope="col"
									class="px-6 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Tanggal Laporan</th
								>
								<th scope="col" class="relative px-6 py-3"><span class="sr-only">Aksi</span></th>
							</tr>
						</thead>
						<tbody class="divide-y divide-gray-200 bg-white">
							{#each filteredData as item}
								<tr class="hover:bg-gray-50">
									<td class="px-6 py-4 whitespace-nowrap">
										<div class="flex items-center">
											<div class="max-w-xs truncate text-sm font-medium text-gray-900">
												{item.url}
											</div>
										</div>
									</td>
									<td class="px-6 py-4 whitespace-nowrap">
										<span
											class="inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium
											{item.status === 'approved'
												? 'bg-green-100 text-green-800'
												: item.status === 'pending'
													? 'bg-yellow-100 text-yellow-800'
													: 'bg-red-100 text-red-800'}"
										>
											{item.status}
										</span>
									</td>
									<td class="px-6 py-4 whitespace-nowrap">
										<div class="text-sm text-gray-500">{item.target}</div>
									</td>
									<td class="px-6 py-4 whitespace-nowrap">
										<div class="text-sm text-gray-500">{item.reporter}</div>
									</td>
									<td class="px-6 py-4 text-sm whitespace-nowrap text-gray-500">
										{item.created_at
											? new Date(item.created_at).toLocaleDateString('id-ID', {
													year: 'numeric',
													month: '2-digit',
													day: '2-digit'
												})
											: 'Tidak ada tanggal'}
									</td>
									<td class="px-6 py-4 text-right text-sm font-medium whitespace-nowrap">
										<a href={`/detail/${item.id}`} class="text-blue-600 hover:text-blue-900"
											>Detail</a
										>
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			{/if}
		</div>

		<!-- Pagination -->
		<div
			class="mt-4 flex flex-col items-center justify-between gap-3 px-2 sm:mt-6 sm:flex-row sm:gap-0 sm:px-4"
		>
			<div class="text-xs text-gray-600 sm:text-sm">
				Menampilkan {Math.min((currentPage - 1) * itemsPerPage + 1, filteredData.length)}-{Math.min(
					currentPage * itemsPerPage,
					filteredData.length
				)} dari {filteredData.length} entri
			</div>
			<div class="flex space-x-1 sm:space-x-2">
				<button
					class="rounded-lg border px-2 py-1 text-xs text-gray-600 hover:bg-gray-50 disabled:opacity-50 sm:px-3 sm:py-1 sm:text-sm"
					disabled={currentPage === 1}
					on:click={() => goToPage(currentPage - 1)}
				>
					Sebelumnya
				</button>

				{#if totalPages > 1}
					{#each { length: totalPages } as _, i}
						<button
							class={`rounded-lg border px-2 py-1 text-xs sm:px-3 sm:py-1 sm:text-sm ${
								currentPage === i + 1 ? 'bg-blue-600 text-white' : 'text-gray-600 hover:bg-gray-50'
							}`}
							on:click={() => goToPage(i + 1)}
						>
							{i + 1}
						</button>
					{/each}
				{:else}
					<button
						class="rounded-lg border bg-blue-600 px-2 py-1 text-xs text-white sm:px-3 sm:py-1 sm:text-sm"
						disabled
					>
						1
					</button>
				{/if}

				<button
					class="rounded-lg border px-2 py-1 text-xs text-gray-600 hover:bg-gray-50 disabled:opacity-50 sm:px-3 sm:py-1 sm:text-sm"
					disabled={currentPage === totalPages}
					on:click={() => goToPage(currentPage + 1)}
				>
					Selanjutnya
				</button>
			</div>
		</div>
	</main>
</div>
