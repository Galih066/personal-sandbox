<script lang="ts">
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { PUBLIC_SESSION_KEY } from '$env/static/public';

	let { children } = $props();

	// Sidebar open state — persisted per-session
	let sidebarOpen = $state(true);
	let mobileOpen = $state(false);

	const navItems = [
		{
			label: 'Dashboard',
			href: '/dashboard',
			icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg>`
		},
		{
			label: 'Projects',
			href: '/dashboard/projects',
			icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M3 7a2 2 0 0 1 2-2h4l2 2h8a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V7z"/></svg>`
		},
		{
			label: 'Notes',
			href: '/dashboard/notes',
			icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14,2 14,8 20,8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><line x1="10" y1="9" x2="8" y2="9"/></svg>`
		},
		{
			label: 'Analytics',
			href: '/dashboard/analytics',
			icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>`
		},
		{
			label: 'Settings',
			href: '/dashboard/settings',
			icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/></svg>`
		}
	];

	function isActive(href: string) {
		return $page.url.pathname === href;
	}

	function closeMobile() {
		mobileOpen = false;
	}

	function logout() {
		sessionStorage.removeItem(PUBLIC_SESSION_KEY);
		sessionStorage.removeItem('psb_token');
		goto('/');
	}
</script>

<div
	class="flex h-screen overflow-hidden bg-[#090c14]"
	style="font-family: 'Inter', system-ui, sans-serif;"
>
	<!-- ── Mobile backdrop ──────────────────────────────── -->
	{#if mobileOpen}
		<button
			class="fixed inset-0 z-20 bg-black/60 backdrop-blur-sm lg:hidden"
			onclick={closeMobile}
			aria-label="Close sidebar"
		></button>
	{/if}

	<!-- ── Sidebar ───────────────────────────────────────── -->
	<aside
		class="fixed inset-y-0 left-0 z-30 flex w-64 flex-col border-r border-white/[0.06] bg-[#0d1117] transition-transform duration-300 ease-in-out
			lg:relative lg:translate-x-0
			{mobileOpen ? 'translate-x-0' : '-translate-x-full'}"
	>
		<!-- Brand -->
		<div class="flex h-16 items-center gap-3 border-b border-white/[0.06] px-5">
			<div
				class="flex h-8 w-8 items-center justify-center rounded-lg text-white"
				style="background: linear-gradient(135deg, #6366f1, #8b5cf6);"
			>
				<svg
					class="h-4 w-4"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2.5"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2" />
				</svg>
			</div>
			<span class="text-sm font-semibold tracking-wide text-slate-100">Personal Sandbox</span>
		</div>

		<!-- Nav -->
		<nav class="flex-1 overflow-y-auto px-3 py-4">
			<p class="mb-2 px-2 text-[0.65rem] font-semibold tracking-widest text-slate-500 uppercase">
				Menu
			</p>
			<ul class="space-y-0.5">
				{#each navItems as item}
					<li>
						<a
							href={item.href}
							onclick={closeMobile}
							class="group flex items-center gap-3 rounded-lg px-3 py-2.5 text-sm font-medium transition-all duration-150
								{isActive(item.href)
								? 'bg-indigo-500/15 text-indigo-400'
								: 'text-slate-400 hover:bg-white/[0.05] hover:text-slate-100'}"
						>
							<span
								class="h-[18px] w-[18px] shrink-0 {isActive(item.href)
									? 'text-indigo-400'
									: 'text-slate-500 group-hover:text-slate-300'}">{@html item.icon}</span
							>
							{item.label}

							{#if isActive(item.href)}
								<span class="ml-auto h-1.5 w-1.5 rounded-full bg-indigo-400"></span>
							{/if}
						</a>
					</li>
				{/each}
			</ul>
		</nav>

		<!-- User profile + logout -->
		<div class="space-y-1 border-t border-white/[0.06] p-3">
			<div class="flex items-center gap-3 rounded-lg px-2 py-2.5">
				<div
					class="flex h-8 w-8 shrink-0 items-center justify-center rounded-full text-xs font-bold text-white"
					style="background: linear-gradient(135deg, #6366f1, #8b5cf6);"
				>
					G
				</div>
				<div class="min-w-0 flex-1">
					<p class="truncate text-xs font-semibold text-slate-200">Galih Dapa Imanda</p>
					<p class="truncate text-[0.65rem] text-slate-500">Personal</p>
				</div>
			</div>
			<button
				onclick={logout}
				class="flex w-full cursor-pointer items-center gap-3 rounded-lg px-3 py-2 text-sm font-medium text-slate-500 transition-colors duration-150 hover:bg-red-500/10 hover:text-red-400"
			>
				<svg
					class="h-4 w-4 shrink-0"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4" />
					<polyline points="16 17 21 12 16 7" />
					<line x1="21" y1="12" x2="9" y2="12" />
				</svg>
				Sign out
			</button>
		</div>
	</aside>

	<!-- ── Main area ─────────────────────────────────────── -->
	<div class="flex min-w-0 flex-1 flex-col overflow-hidden">
		<!-- Top bar -->
		<header
			class="flex h-16 shrink-0 items-center gap-4 border-b border-white/[0.06] bg-[#0d1117] px-4 lg:px-6"
		>
			<!-- Hamburger (mobile only) -->
			<button
				class="flex h-9 w-9 items-center justify-center rounded-lg text-slate-400 transition-colors hover:bg-white/[0.06] hover:text-slate-100 lg:hidden"
				onclick={() => (mobileOpen = !mobileOpen)}
				aria-label="Toggle sidebar"
			>
				<svg
					class="h-5 w-5"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
				>
					<line x1="3" y1="6" x2="21" y2="6" />
					<line x1="3" y1="12" x2="21" y2="12" />
					<line x1="3" y1="18" x2="21" y2="18" />
				</svg>
			</button>

			<!-- Desktop sidebar toggle -->
			<button
				class="hidden h-9 w-9 items-center justify-center rounded-lg text-slate-400 transition-colors hover:bg-white/[0.06] hover:text-slate-100 lg:flex"
				onclick={() => (sidebarOpen = !sidebarOpen)}
				aria-label="Toggle sidebar"
			>
				<svg
					class="h-5 w-5"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
				>
					<line x1="3" y1="6" x2="21" y2="6" />
					<line x1="3" y1="12" x2="21" y2="12" />
					<line x1="3" y1="18" x2="21" y2="18" />
				</svg>
			</button>

			<!-- Breadcrumb -->
			<div class="flex items-center gap-2 text-sm">
				<span class="text-slate-500">Sandbox</span>
				<span class="text-slate-700">/</span>
				<span class="font-medium text-slate-200 capitalize">
					{$page.url.pathname.split('/').filter(Boolean).pop() ?? 'Dashboard'}
				</span>
			</div>

			<!-- Right side -->
			<div class="ml-auto flex items-center gap-2">
				<!-- Search -->
				<div
					class="hidden items-center gap-2 rounded-lg border border-white/[0.08] bg-white/[0.04] px-3 py-1.5 sm:flex"
				>
					<svg
						class="h-4 w-4 text-slate-500"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2"
						stroke-linecap="round"
					>
						<circle cx="11" cy="11" r="8" /><line x1="21" y1="21" x2="16.65" y2="16.65" />
					</svg>
					<input
						type="text"
						placeholder="Search…"
						class="w-32 bg-transparent text-xs text-slate-300 outline-none placeholder:text-slate-600"
					/>
					<kbd class="rounded border border-white/[0.08] px-1 text-[0.6rem] text-slate-600">⌘K</kbd>
				</div>

				<!-- Notifications -->
				<button
					class="relative flex h-9 w-9 items-center justify-center rounded-lg text-slate-400 transition-colors hover:bg-white/[0.06] hover:text-slate-100"
					aria-label="Notifications"
				>
					<svg
						class="h-4.5 w-4.5"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2"
						stroke-linecap="round"
					>
						<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9" /><path
							d="M13.73 21a2 2 0 0 1-3.46 0"
						/>
					</svg>
					<span class="absolute top-1.5 right-1.5 h-2 w-2 rounded-full bg-indigo-500"></span>
				</button>

				<!-- Avatar -->
				<div
					class="flex h-9 w-9 cursor-pointer items-center justify-center rounded-full text-xs font-bold text-white"
					style="background: linear-gradient(135deg, #6366f1, #8b5cf6);"
				>
					G
				</div>
			</div>
		</header>

		<!-- Page content slot -->
		<main class="flex-1 overflow-y-auto p-4 lg:p-6">
			{@render children()}
		</main>
	</div>
</div>
