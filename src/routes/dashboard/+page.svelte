<script lang="ts">
	const stats = [
		{ label: 'Projects', value: '12', delta: '+2 this month', up: true },
		{ label: 'Notes', value: '48', delta: '+5 this week', up: true },
		{ label: 'Commits', value: '236', delta: '+18 today', up: true },
		{ label: 'Uptime', value: '99.9%', delta: 'All systems go', up: true }
	];

	const recentActivity = [
		{ action: 'Pushed to', target: 'personal-sandbox', time: '2 min ago', type: 'commit' },
		{ action: 'Created note', target: 'API design ideas', time: '1 hr ago', type: 'note' },
		{ action: 'Updated', target: 'Dashboard layout', time: '3 hr ago', type: 'edit' },
		{ action: 'Deployed', target: 'recos-art API', time: 'Yesterday', type: 'deploy' },
		{ action: 'Created project', target: 'ML Experiments', time: '2 days ago', type: 'project' }
	];

	const activityIcon: Record<string, string> = {
		commit: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="4"/><line x1="1.05" y1="12" x2="7" y2="12"/><line x1="17.01" y1="12" x2="22.96" y2="12"/></svg>`,
		note: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14,2 14,8 20,8"/></svg>`,
		edit: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></svg>`,
		deploy: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/></svg>`,
		project: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M3 7a2 2 0 0 1 2-2h4l2 2h8a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V7z"/></svg>`
	};

	const activityColor: Record<string, string> = {
		commit: 'bg-indigo-500/15 text-indigo-400',
		note: 'bg-amber-500/15 text-amber-400',
		edit: 'bg-sky-500/15 text-sky-400',
		deploy: 'bg-green-500/15 text-green-400',
		project: 'bg-purple-500/15 text-purple-400'
	};
</script>

<!-- Page header -->
<div class="mb-6">
	<h1 class="text-xl font-bold text-slate-100">Dashboard</h1>
	<p class="mt-0.5 text-sm text-slate-500">Welcome back, sir. Here's what's going on.</p>
</div>

<!-- Stats grid -->
<div class="mb-6 grid grid-cols-2 gap-3 lg:grid-cols-4">
	{#each stats as stat}
		<div
			class="rounded-xl border border-white/[0.06] bg-white/[0.03] p-4 transition-colors hover:bg-white/[0.05]"
		>
			<p class="text-xs font-medium text-slate-500">{stat.label}</p>
			<p class="mt-1.5 text-2xl font-bold text-slate-100">{stat.value}</p>
			<p class="mt-1 text-xs {stat.up ? 'text-green-400' : 'text-red-400'}">{stat.delta}</p>
		</div>
	{/each}
</div>

<!-- Main grid -->
<div class="grid grid-cols-1 gap-4 lg:grid-cols-3">
	<!-- Recent activity -->
	<div class="rounded-xl border border-white/[0.06] bg-white/[0.03] p-4 lg:col-span-2">
		<div class="mb-4 flex items-center justify-between">
			<h2 class="text-sm font-semibold text-slate-200">Recent Activity</h2>
			<button class="text-xs text-indigo-400 hover:text-indigo-300">View all</button>
		</div>
		<ul class="space-y-3">
			{#each recentActivity as item}
				<li class="flex items-center gap-3">
					<span
						class="flex h-8 w-8 shrink-0 items-center justify-center rounded-lg {activityColor[
							item.type
						]}"
					>
						<span class="h-4 w-4">{@html activityIcon[item.type]}</span>
					</span>
					<div class="min-w-0 flex-1">
						<p class="text-sm text-slate-300">
							{item.action} <span class="font-medium text-slate-100">"{item.target}"</span>
						</p>
					</div>
					<span class="shrink-0 text-xs text-slate-600">{item.time}</span>
				</li>
			{/each}
		</ul>
	</div>

	<!-- Quick actions -->
	<div class="rounded-xl border border-white/[0.06] bg-white/[0.03] p-4">
		<h2 class="mb-4 text-sm font-semibold text-slate-200">Quick Actions</h2>
		<div class="space-y-2">
			{#each [{ label: 'New Project', color: 'bg-indigo-500/15 text-indigo-400 hover:bg-indigo-500/25', icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>` }, { label: 'New Note', color: 'bg-amber-500/15 text-amber-400 hover:bg-amber-500/25', icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>` }, { label: 'Open Terminal', color: 'bg-green-500/15 text-green-400 hover:bg-green-500/25', icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="4 17 10 11 4 5"/><line x1="12" y1="19" x2="20" y2="19"/></svg>` }, { label: 'Settings', color: 'bg-sky-500/15 text-sky-400 hover:bg-sky-500/25', icon: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/></svg>` }] as action}
				<button
					class="flex w-full items-center gap-3 rounded-lg px-3 py-2.5 text-sm font-medium transition-colors duration-150 {action.color}"
				>
					<span class="h-4 w-4">{@html action.icon}</span>
					{action.label}
				</button>
			{/each}
		</div>
	</div>
</div>
