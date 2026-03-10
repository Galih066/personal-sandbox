<script lang="ts">
	import { onMount } from 'svelte';
	import { PUBLIC_API_BASE_URL, PUBLIC_EMAIL, PUBLIC_SESSION_KEY } from '$env/static/public';

	const EMAIL = PUBLIC_EMAIL;
	const LOGIN_URL = `${PUBLIC_API_BASE_URL}/api/auth/login`;
	const SESSION_KEY = PUBLIC_SESSION_KEY;

	let digits = $state<string[]>(Array(6).fill(''));
	let inputRefs: HTMLInputElement[] = [];
	let isAuthenticated = $state(false);
	let shaking = $state(false);
	let success = $state(false);
	let loading = $state(false);
	let mounted = $state(false);
	let errorMsg = $state('');

	onMount(() => {
		if (sessionStorage.getItem(SESSION_KEY) === 'true') isAuthenticated = true;
		mounted = true;
		setTimeout(() => inputRefs[0]?.focus(), 100);
	});

	function handleInput(index: number, e: Event) {
		const val = (e.target as HTMLInputElement).value;
		const filtered = val.replace(/\D/g, '').slice(-1);
		digits[index] = filtered;
		(e.target as HTMLInputElement).value = filtered;

		if (filtered && index < 5) inputRefs[index + 1]?.focus();
		if (digits.every((d) => d !== '')) submitPin();
	}

	function handleKeyDown(index: number, e: KeyboardEvent) {
		if (e.key === 'Backspace' && !digits[index] && index > 0) {
			digits[index - 1] = '';
			inputRefs[index - 1]?.focus();
		}
		if (e.key === 'ArrowLeft' && index > 0) inputRefs[index - 1]?.focus();
		if (e.key === 'ArrowRight' && index < 5) inputRefs[index + 1]?.focus();
	}

	function handlePaste(e: ClipboardEvent) {
		e.preventDefault();
		const pasted = e.clipboardData?.getData('text').replace(/\D/g, '').slice(0, 6) ?? '';
		for (let i = 0; i < 6; i++) digits[i] = pasted[i] ?? '';
		if (pasted.length === 6) submitPin();
		else inputRefs[Math.min(pasted.length, 5)]?.focus();
	}

	async function submitPin() {
		if (loading) return;
		loading = true;
		errorMsg = '';

		const password = digits.join('');

		try {
			const res = await fetch(LOGIN_URL, {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ email: EMAIL, password })
			});

			const data = await res.json();

			if (res.ok) {
				if (data?.token) sessionStorage.setItem('psb_token', data.token);
				if (data?.data?.token) sessionStorage.setItem('psb_token', data.data.token);
				sessionStorage.setItem(SESSION_KEY, 'true');
				success = true;
				setTimeout(() => (isAuthenticated = true), 800);
			} else {
				triggerError(data?.message ?? 'Incorrect PIN, try again');
			}
		} catch {
			triggerError('Network error — check your connection');
		} finally {
			loading = false;
		}
	}

	function triggerError(msg: string) {
		errorMsg = msg;
		shaking = true;
		setTimeout(() => {
			shaking = false;
			digits = Array(6).fill('');
			inputRefs.forEach((el) => (el.value = ''));
			inputRefs[0]?.focus();
		}, 600);
	}
</script>

{#if !mounted}
	<!-- prevent flash -->
{:else if isAuthenticated}
	<!-- ── Authenticated ───────────────────────────── -->
	<div
		class="flex min-h-screen items-center justify-center bg-[#090c14]"
		style="font-family: 'Inter', system-ui, sans-serif;"
	>
		<div class="text-center" style="animation: fadein 0.5s ease;">
			<div
				class="mb-4 text-5xl text-green-400"
				style="animation: pop 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);"
			>
				✓
			</div>
			<h1 class="mb-2 text-[2rem] font-bold tracking-tight text-slate-100">Welcome back, sir!</h1>
			<p class="mb-8 text-[0.9375rem] text-slate-500">You're in. The sandbox is yours.</p>
			<a
				href="/dashboard"
				class="inline-block rounded-xl px-8 py-3 text-[0.9375rem] font-semibold text-white no-underline transition-all duration-150 hover:-translate-y-0.5"
				style="background: linear-gradient(135deg, #6366f1, #8b5cf6); box-shadow: 0 8px 24px rgba(99,102,241,0.4);"
				>Enter ↗</a
			>
		</div>
	</div>
{:else}
	<!-- ── Login page ───────────────────────────────── -->
	<div
		class="relative flex min-h-screen flex-col items-center justify-center overflow-hidden bg-[#090c14]"
		style="font-family: 'Inter', system-ui, sans-serif;"
	>
		<!-- Background blobs -->
		<div
			class="pointer-events-none absolute h-[400px] w-[400px] rounded-full opacity-[0.18]"
			style="filter: blur(80px); background: radial-gradient(circle, #6366f1, #4f46e5); top: -120px; left: -100px; animation: float 8s ease-in-out infinite alternate;"
		></div>
		<div
			class="pointer-events-none absolute h-[350px] w-[350px] rounded-full opacity-[0.18]"
			style="filter: blur(80px); background: radial-gradient(circle, #8b5cf6, #7c3aed); bottom: -100px; right: -80px; animation: float 8s ease-in-out -3s infinite alternate;"
		></div>
		<div
			class="pointer-events-none absolute h-[250px] w-[250px] rounded-full opacity-[0.18]"
			style="filter: blur(80px); background: radial-gradient(circle, #06b6d4, #0284c7); top: 50%; left: 60%; animation: float 8s ease-in-out -6s infinite alternate;"
		></div>

		<!-- Card -->
		<div
			class="relative z-10 w-[min(480px,calc(100vw-32px))] border border-white/10 bg-white/[0.04] backdrop-blur-xl"
			class:shaking
			style="border-radius: 24px; box-shadow: 0 0 0 1px rgba(255,255,255,0.05), 0 24px 64px rgba(0,0,0,0.5), inset 0 1px 0 rgba(255,255,255,0.08);"
		>
			<div class="px-10 pt-12 pb-10">
				<!-- Header -->
				<div class="mb-10 text-center">
					<div
						class="mx-auto mb-5 flex h-16 w-16 items-center justify-center rounded-full text-white"
						class:avatar-loading={loading}
						style="background: linear-gradient(135deg, #6366f1, #8b5cf6); box-shadow: 0 0 0 4px rgba(99,102,241,0.2), 0 8px 24px rgba(99,102,241,0.4);"
					>
						{#if loading}
							<svg class="spinner h-8 w-8" viewBox="0 0 24 24" fill="none">
								<circle
									cx="12"
									cy="12"
									r="10"
									stroke="currentColor"
									stroke-width="2"
									opacity="0.25"
								/>
								<path
									d="M12 2a10 10 0 0 1 10 10"
									stroke="currentColor"
									stroke-width="2"
									stroke-linecap="round"
								/>
							</svg>
						{:else}
							<svg class="h-8 w-8" viewBox="0 0 40 40" fill="none">
								<circle cx="20" cy="15" r="7" stroke="currentColor" stroke-width="2" />
								<path
									d="M6 36c0-7.732 6.268-14 14-14s14 6.268 14 14"
									stroke="currentColor"
									stroke-width="2"
									stroke-linecap="round"
								/>
							</svg>
						{/if}
					</div>
					<h1 class="mb-2.5 text-[1.75rem] leading-tight font-bold tracking-tight text-slate-100">
						Welcome back, sir!
					</h1>
					<p class="text-sm leading-relaxed text-slate-400">
						Enter your 6-digit PIN to access your personal space.
					</p>
				</div>

				<!-- PIN boxes -->
				<div class="mb-5 flex justify-center gap-2.5">
					{#each digits as digit, i}
						<input
							bind:this={inputRefs[i]}
							type="tel"
							inputmode="numeric"
							pattern="[0-9]*"
							maxlength="1"
							value={digit}
							class="pin-box h-16 w-14 text-center text-2xl font-bold transition-all duration-200 outline-none disabled:cursor-not-allowed disabled:opacity-50
								{success
								? 'border-green-500 text-green-400'
								: digit !== ''
									? 'pin-box--filled text-slate-100'
									: 'text-slate-100'}"
							disabled={loading}
							aria-label={`Digit ${i + 1}`}
							oninput={(e) => handleInput(i, e)}
							onkeydown={(e) => handleKeyDown(i, e)}
							onpaste={handlePaste}
							onfocus={(e) => (e.target as HTMLInputElement).select()}
						/>
					{/each}
				</div>

				<!-- Hint -->
				<p class="m-0 min-h-5 text-center text-[0.8125rem]">
					{#if success}
						<span class="text-green-400" style="animation: fadein 0.2s ease;">✓ Access granted</span
						>
					{:else if loading}
						<span class="text-indigo-400">Verifying…</span>
					{:else if shaking}
						<span class="text-red-400" style="animation: fadein 0.2s ease;">✗ {errorMsg}</span>
					{:else}
						<span class="text-slate-600">Private access only</span>
					{/if}
				</p>
			</div>
		</div>

		<p class="relative z-10 mt-7 text-xs tracking-widest text-slate-700">
			© Personal Sandbox · {new Date().getFullYear()}
		</p>
	</div>
{/if}

<style>
	/* Only truly non-Tailwindable styles + keyframes live here */

	.pin-box {
		border: 1.5px solid rgba(255, 255, 255, 0.12);
		background: rgba(255, 255, 255, 0.05);
		border-radius: 14px;
		caret-color: transparent;
		-webkit-text-security: disc;
		font-family: inherit;
	}
	.pin-box--filled {
		border-color: rgba(99, 102, 241, 0.5);
		background: rgba(99, 102, 241, 0.06);
	}
	.pin-box:focus:not(:disabled) {
		border-color: #6366f1;
		background: rgba(99, 102, 241, 0.08);
		box-shadow:
			0 0 0 3px rgba(99, 102, 241, 0.2),
			0 4px 12px rgba(0, 0, 0, 0.3);
		transform: translateY(-2px);
	}
	.pin-box.border-green-500 {
		background: rgba(34, 197, 94, 0.1);
		box-shadow: 0 0 12px rgba(34, 197, 94, 0.3);
	}

	.shaking {
		animation: shake 0.5s cubic-bezier(0.36, 0.07, 0.19, 0.97);
	}
	.avatar-loading {
		animation: pulse-ring 1s ease infinite;
	}
	.spinner {
		animation: spin 0.8s linear infinite;
	}

	@keyframes float {
		from {
			transform: translate(0, 0) scale(1);
		}
		to {
			transform: translate(30px, -30px) scale(1.1);
		}
	}
	@keyframes shake {
		10%,
		90% {
			transform: translateX(-4px);
		}
		20%,
		80% {
			transform: translateX(6px);
		}
		30%,
		50%,
		70% {
			transform: translateX(-8px);
		}
		40%,
		60% {
			transform: translateX(8px);
		}
	}
	@keyframes fadein {
		from {
			opacity: 0;
			transform: translateY(4px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	@keyframes pop {
		from {
			transform: scale(0);
			opacity: 0;
		}
		to {
			transform: scale(1);
			opacity: 1;
		}
	}
	@keyframes pulse-ring {
		0% {
			box-shadow:
				0 0 0 4px rgba(99, 102, 241, 0.2),
				0 8px 24px rgba(99, 102, 241, 0.4);
		}
		50% {
			box-shadow:
				0 0 0 8px rgba(99, 102, 241, 0.1),
				0 8px 32px rgba(99, 102, 241, 0.6);
		}
		100% {
			box-shadow:
				0 0 0 4px rgba(99, 102, 241, 0.2),
				0 8px 24px rgba(99, 102, 241, 0.4);
		}
	}
	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}

	@media (max-width: 400px) {
		.pin-box {
			width: 42px;
			height: 52px;
			font-size: 1.25rem;
		}
	}
</style>
