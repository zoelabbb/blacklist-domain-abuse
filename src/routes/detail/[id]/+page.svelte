<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/stores';

	let id: string;
	let detailData: any = null;
	let isLoading = true;
	let error: string | null = null;

	const API_URL = import.meta.env.VITE_API_URL;
	const API_KEY = import.meta.env.VITE_API_KEY;

	// Get the ID from the URL
	id = $page.params.id;

	async function fetchDetail() {
		try {
			const res = await fetch(`${API_URL}/api/v1/data-phishing/${id}`, {
				method: 'GET',
				headers: {
					'X-API-KEY': API_KEY,
					'Content-Type': 'application/json'
				}
			});

			if (!res.ok) {
				throw new Error('Gagal mengambil detail data');
			}

			const response = await res.json();
			detailData = response.data;
		} catch (err) {
			error = (err as Error).message;
			console.error('Error fetching detail:', err);
		} finally {
			isLoading = false;
		}
	}

	onMount(async () => {
		await fetchDetail();
	});
</script>

<svelte:head>
	<title>Detail Blacklist - {id}</title>
	<meta name="description" content="Detail laporan blacklist" />
</svelte:head>

<div class="min-h-screen bg-gray-50 px-4 py-8 sm:px-6 lg:px-8">
	<main class="mx-auto max-w-4xl space-y-6">
		<!-- Header with back button -->
		<div class="flex items-center">
			<a
				href="/"
				class="mr-4 flex items-center text-gray-600 transition-colors hover:text-gray-900"
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-5 w-5"
					viewBox="0 0 20 20"
					fill="currentColor"
				>
					<path
						fill-rule="evenodd"
						d="M9.707 16.707a1 1 0 01-1.414 0l-6-6a1 1 0 010-1.414l6-6a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l4.293 4.293a1 1 0 010 1.414z"
						clip-rule="evenodd"
					/>
				</svg>
				<span class="ml-1 text-sm font-medium">Kembali</span>
			</a>
			<h1 class="text-xl font-bold text-gray-900">Detail Laporan</h1>
		</div>

		<!-- Loading State -->
		{#if isLoading}
			<div class="flex justify-center py-12">
				<div
					class="h-12 w-12 animate-spin rounded-full border-4 border-blue-500 border-t-transparent"
				></div>
			</div>
		{:else if error}
			<div class="rounded-lg bg-red-50 p-4">
				<div class="flex">
					<div class="flex-shrink-0">
						<svg
							class="h-5 w-5 text-red-400"
							xmlns="http://www.w3.org/2000/svg"
							viewBox="0 0 20 20"
							fill="currentColor"
						>
							<path
								fill-rule="evenodd"
								d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z"
								clip-rule="evenodd"
							/>
						</svg>
					</div>
					<div class="ml-3">
						<h3 class="text-sm font-medium text-red-800">Error</h3>
						<div class="mt-2 text-sm text-red-700">
							<p>{error}</p>
						</div>
					</div>
				</div>
			</div>
		{:else}
			<!-- Detail Content -->
			<div class="overflow-hidden rounded-xl bg-white shadow">
				<!-- Header Section -->
				<div class="border-b border-gray-200 bg-gray-50 px-6 py-4">
					<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between">
						<div>
							<h2 class="truncate text-lg font-semibold text-gray-900">{detailData.url}</h2>
							<p class="mt-1 text-sm text-gray-500">Laporan ID: {id}</p>
						</div>
						<div class="mt-2 sm:mt-0">
							<span
								class="inline-flex items-center rounded-full px-3 py-1 text-sm font-medium
								{detailData.status === 'approved'
									? 'bg-green-100 text-green-800'
									: detailData.status === 'pending'
										? 'bg-yellow-100 text-yellow-800'
										: 'bg-red-100 text-red-800'}"
							>
								{detailData.status}
							</span>
						</div>
					</div>
				</div>

				<!-- Main Content -->
				<div class="px-6 py-5">
					<div class="grid grid-cols-1 gap-6 sm:grid-cols-2">
						<div class="space-y-4">
							<div>
								<h3 class="text-xs font-medium tracking-wider text-gray-500 uppercase">Target</h3>
								<p class="mt-1 text-sm font-medium text-gray-900">{detailData.target}</p>
							</div>
							<div>
								<h3 class="text-xs font-medium tracking-wider text-gray-500 uppercase">Pelapor</h3>
								<p class="mt-1 text-sm font-medium text-gray-900">{detailData.reporter}</p>
							</div>
						</div>
						<div class="space-y-4">
							<div>
								<h3 class="text-xs font-medium tracking-wider text-gray-500 uppercase">
									Tanggal Laporan
								</h3>
								<p class="mt-1 text-sm font-medium text-gray-900">
									{detailData.created_at.split('T')[0]}
								</p>
							</div>
							<div>
								<h3 class="text-xs font-medium tracking-wider text-gray-500 uppercase">
									Terakhir Diupdate
								</h3>
								<p class="mt-1 text-sm font-medium text-gray-900">
									{detailData.updated_at ? detailData.updated_at.split('T')[0] : 'N/A'}
								</p>
							</div>
						</div>
					</div>

					<div class="mt-8">
						<h3 class="text-xs font-medium tracking-wider text-gray-500 uppercase">Deskripsi</h3>
						<div class="mt-2 rounded-lg bg-gray-50 p-4">
							<p class="text-sm text-gray-700">
								{detailData.description || 'Tidak ada deskripsi tersedia.'}
							</p>
						</div>
					</div>

					<div class="mt-8 flex justify-end">
						<a
							href="/"
							class="inline-flex items-center rounded-md border border-transparent bg-blue-600 px-4 py-2 text-sm font-medium text-white shadow-sm transition-colors hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 focus:outline-none"
						>
							Kembali ke Daftar
						</a>
					</div>
				</div>
			</div>
		{/if}
	</main>
</div>
