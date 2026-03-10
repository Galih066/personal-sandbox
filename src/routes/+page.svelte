<script lang="ts">
	import { onMount } from 'svelte';
	import { PUBLIC_API_BASE_URL, PUBLIC_SESSION_KEY } from '$env/static/public';

	// ── CONFIG ──────────────────────────────────────────────
	const EMAIL = 'galihdapaimanda@gmail.com';
	const LOGIN_URL = `${PUBLIC_API_BASE_URL}/api/auth/login`;
	const SESSION_KEY = PUBLIC_SESSION_KEY;
	// ────────────────────────────────────────────────────────

	let digits = $state<string[]>(Array(6).fill(''));
	let inputRefs: HTMLInputElement[] = [];
	let isAuthenticated = $state(false);
	let shaking = $state(false);
	let success = $state(false);
	let loading = $state(false);
	let mounted = $state(false);
	let errorMsg = $state('');

	onMount(() => {
		if (sessionStorage.getItem(SESSION_KEY) === 'true') {
			isAuthenticated = true;
		}
		mounted = true;
		setTimeout(() => inputRefs[0]?.focus(), 100);
	});

	function handleInput(index: number, e: Event) {
		const val = (e.target as HTMLInputElement).value;
		const filtered = val.replace(/\D/g, '').slice(-1);
		digits[index] = filtered;
		(e.target as HTMLInputElement).value = filtered;

		if (filtered && index < 5) {
			inputRefs[index + 1]?.focus();
		}
		if (digits.every((d) => d !== '')) {
			submitPin();
		}
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
		for (let i = 0; i < 6; i++) {
			digits[i] = pasted[i] ?? '';
		}
		if (pasted.length === 6) {
			submitPin();
		} else {
			inputRefs[Math.min(pasted.length, 5)]?.focus();
		}
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
				// Store token if returned by the API
				if (data?.token) sessionStorage.setItem('psb_token', data.token);
				if (data?.data?.token) sessionStorage.setItem('psb_token', data.data.token);

				sessionStorage.setItem(SESSION_KEY, 'true');
				success = true;
				setTimeout(() => {
					isAuthenticated = true;
				}, 800);
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
	<div class="authenticated">
		<div class="auth-content">
			<div class="checkmark">✓</div>
			<h1>Welcome back, sir!</h1>
			<p class="sub">You're in. The sandbox is yours.</p>
			<a href="/dashboard" class="btn-enter">Enter ↗</a>
		</div>
	</div>
{:else}
	<div class="page">
		<!-- Animated background blobs -->
		<div class="blob blob-1"></div>
		<div class="blob blob-2"></div>
		<div class="blob blob-3"></div>

		<div class="card" class:shaking>
			<div class="card-inner">
				<!-- Header -->
				<div class="header">
					<div class="avatar" class:loading>
						{#if loading}
							<svg
								class="spinner"
								viewBox="0 0 24 24"
								fill="none"
								xmlns="http://www.w3.org/2000/svg"
							>
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
							<svg viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
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
					<h1 class="title">Welcome back, sir!</h1>
					<p class="subtitle">Enter your 6-digit PIN to access your personal space.</p>
				</div>

				<!-- PIN Input -->
				<div class="pin-wrap" class:success>
					{#each digits as digit, i}
						<input
							bind:this={inputRefs[i]}
							type="tel"
							inputmode="numeric"
							pattern="[0-9]*"
							maxlength="1"
							value={digit}
							class="pin-box"
							class:filled={digit !== ''}
							class:success
							disabled={loading}
							aria-label={`Digit ${i + 1}`}
							oninput={(e) => handleInput(i, e)}
							onkeydown={(e) => handleKeyDown(i, e)}
							onpaste={handlePaste}
							onfocus={(e) => (e.target as HTMLInputElement).select()}
						/>
					{/each}
				</div>

				<p class="hint">
					{#if success}
						<span class="hint-success">✓ Access granted</span>
					{:else if loading}
						<span class="hint-loading">Verifying…</span>
					{:else if shaking}
						<span class="hint-error">✗ {errorMsg}</span>
					{:else}
						<span class="hint-neutral">Private access only</span>
					{/if}
				</p>
			</div>
		</div>

		<p class="footer-note">© Personal Sandbox · {new Date().getFullYear()}</p>
	</div>
{/if}

<style>
	/* ── Page ─────────────────────────────────────────── */
	.page {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background: #090c14;
		position: relative;
		overflow: hidden;
		font-family:
			'Inter',
			system-ui,
			-apple-system,
			sans-serif;
	}

	/* ── Animated blobs ───────────────────────────────── */
	.blob {
		position: absolute;
		border-radius: 50%;
		filter: blur(80px);
		opacity: 0.18;
		animation: float 8s ease-in-out infinite alternate;
		pointer-events: none;
	}
	.blob-1 {
		width: 400px;
		height: 400px;
		background: radial-gradient(circle, #6366f1, #4f46e5);
		top: -120px;
		left: -100px;
		animation-delay: 0s;
	}
	.blob-2 {
		width: 350px;
		height: 350px;
		background: radial-gradient(circle, #8b5cf6, #7c3aed);
		bottom: -100px;
		right: -80px;
		animation-delay: -3s;
	}
	.blob-3 {
		width: 250px;
		height: 250px;
		background: radial-gradient(circle, #06b6d4, #0284c7);
		top: 50%;
		left: 60%;
		animation-delay: -6s;
	}
	@keyframes float {
		from {
			transform: translate(0, 0) scale(1);
		}
		to {
			transform: translate(30px, -30px) scale(1.1);
		}
	}

	/* ── Card ─────────────────────────────────────────── */
	.card {
		position: relative;
		z-index: 10;
		background: rgba(255, 255, 255, 0.04);
		border: 1px solid rgba(255, 255, 255, 0.1);
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		border-radius: 24px;
		width: min(480px, calc(100vw - 32px));
		box-shadow:
			0 0 0 1px rgba(255, 255, 255, 0.05),
			0 24px 64px rgba(0, 0, 0, 0.5),
			inset 0 1px 0 rgba(255, 255, 255, 0.08);
	}
	.card.shaking {
		animation: shake 0.5s cubic-bezier(0.36, 0.07, 0.19, 0.97);
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
	.card-inner {
		padding: 48px 40px 40px;
	}

	/* ── Header ───────────────────────────────────────── */
	.header {
		text-align: center;
		margin-bottom: 40px;
	}
	.avatar {
		width: 64px;
		height: 64px;
		margin: 0 auto 20px;
		background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		color: white;
		box-shadow:
			0 0 0 4px rgba(99, 102, 241, 0.2),
			0 8px 24px rgba(99, 102, 241, 0.4);
		transition: transform 0.3s ease;
	}
	.avatar.loading {
		animation: pulse-ring 1s ease infinite;
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
	.avatar svg {
		width: 32px;
		height: 32px;
	}
	.spinner {
		animation: spin 0.8s linear infinite;
	}
	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}

	.title {
		font-size: 1.75rem;
		font-weight: 700;
		color: #f1f5f9;
		letter-spacing: -0.02em;
		margin: 0 0 10px;
		line-height: 1.2;
	}
	.subtitle {
		font-size: 0.875rem;
		color: #94a3b8;
		margin: 0;
		line-height: 1.6;
	}

	/* ── PIN boxes ────────────────────────────────────── */
	.pin-wrap {
		display: flex;
		gap: 10px;
		justify-content: center;
		margin-bottom: 20px;
	}
	.pin-wrap.success .pin-box {
		border-color: #22c55e;
		background: rgba(34, 197, 94, 0.1);
		color: #22c55e;
		box-shadow: 0 0 12px rgba(34, 197, 94, 0.3);
	}
	.pin-box {
		width: 56px;
		height: 64px;
		border-radius: 14px;
		border: 1.5px solid rgba(255, 255, 255, 0.12);
		background: rgba(255, 255, 255, 0.05);
		color: #f1f5f9;
		font-size: 1.5rem;
		font-weight: 700;
		text-align: center;
		outline: none;
		caret-color: transparent;
		transition:
			border-color 0.2s,
			background 0.2s,
			box-shadow 0.2s,
			transform 0.15s;
		-webkit-text-security: disc;
		font-family: inherit;
	}
	.pin-box:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}
	.pin-box:focus:not(:disabled) {
		border-color: #6366f1;
		background: rgba(99, 102, 241, 0.08);
		box-shadow:
			0 0 0 3px rgba(99, 102, 241, 0.2),
			0 4px 12px rgba(0, 0, 0, 0.3);
		transform: translateY(-2px);
	}
	.pin-box.filled {
		border-color: rgba(99, 102, 241, 0.5);
		background: rgba(99, 102, 241, 0.06);
	}

	/* ── Hint text ────────────────────────────────────── */
	.hint {
		text-align: center;
		font-size: 0.8125rem;
		min-height: 20px;
		margin: 0;
	}
	.hint-neutral {
		color: #475569;
	}
	.hint-loading {
		color: #818cf8;
	}
	.hint-error {
		color: #f87171;
		animation: fadein 0.2s ease;
	}
	.hint-success {
		color: #22c55e;
		animation: fadein 0.2s ease;
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

	/* ── Footer ───────────────────────────────────────── */
	.footer-note {
		position: relative;
		z-index: 10;
		margin-top: 28px;
		font-size: 0.75rem;
		color: #334155;
		letter-spacing: 0.04em;
	}

	/* ── Authenticated state ──────────────────────────── */
	.authenticated {
		min-height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #090c14;
		font-family: 'Inter', system-ui, sans-serif;
	}
	.auth-content {
		text-align: center;
		animation: fadein 0.5s ease;
	}
	.checkmark {
		font-size: 3rem;
		color: #22c55e;
		margin-bottom: 16px;
		animation: pop 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
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
	.auth-content h1 {
		font-size: 2rem;
		font-weight: 700;
		color: #f1f5f9;
		letter-spacing: -0.02em;
		margin: 0 0 8px;
	}
	.auth-content .sub {
		color: #64748b;
		font-size: 0.9375rem;
		margin: 0 0 32px;
	}
	.btn-enter {
		display: inline-block;
		padding: 12px 32px;
		background: linear-gradient(135deg, #6366f1, #8b5cf6);
		color: white;
		border-radius: 12px;
		font-weight: 600;
		font-size: 0.9375rem;
		text-decoration: none;
		box-shadow: 0 8px 24px rgba(99, 102, 241, 0.4);
		transition:
			transform 0.15s ease,
			box-shadow 0.15s ease;
	}
	.btn-enter:hover {
		transform: translateY(-2px);
		box-shadow: 0 12px 32px rgba(99, 102, 241, 0.5);
	}

	/* ── Responsive ───────────────────────────────────── */
	@media (max-width: 400px) {
		.card-inner {
			padding: 36px 20px 32px;
		}
		.pin-box {
			width: 42px;
			height: 52px;
			font-size: 1.25rem;
		}
		.pin-wrap {
			gap: 6px;
		}
	}
</style>
