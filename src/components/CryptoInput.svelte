<script lang="ts">
	import { onMount } from 'svelte';

	// Props
	export let value: string = '';
	export let currency: 'ETH' | 'BTC' | 'USDT' | 'IDR' = 'USDT';

	// Types
	type CurrencyDecimals = {
		[key in typeof currency]: number;
	};

	// Constants
	const MAX_DECIMALS: CurrencyDecimals = {
		ETH: 18,
		BTC: 8,
		USDT: 6,
		IDR: 2,
	};

	// State
	let inputElement: HTMLInputElement;
	let error: string = '';
	let userFormat: '.' | ',' = '.';

	onMount(() => {
		userFormat = (localStorage.getItem('decimalSeparator') || '.') as '.' | ',';
	});

	function saveUserFormat(format: '.' | ',') {
		userFormat = format;
		localStorage.setItem('decimalSeparator', format);
	}

	function validateInput(value: string): boolean {
		if (!value) return true;

		const parts = value.split(/[.,]/);
		if (parts.length > 1 && parts[1].length > MAX_DECIMALS[currency]) {
			error = `Maximum ${MAX_DECIMALS[currency]} decimals allowed for ${currency}`;
			return false;
		}
		error = '';
		return true;
	}

	function formatNumber(inputValue: string): string {
		if (!inputValue) return '';

		// Remove all non-numeric characters except . and ,
		let cleaned = inputValue.replace(/[^\d.,]/g, '');

		// Determine decimal separator
		const lastDot = cleaned.lastIndexOf('.');
		const lastComma = cleaned.lastIndexOf(',');
		const decimalSeparator = lastDot > lastComma ? '.' : ',';

		saveUserFormat(decimalSeparator as '.' | ',');

		// Split number into parts
		const parts = cleaned.split(decimalSeparator);
		let integerPart = parts[0].replace(/[.,]/g, '');
		let decimalPart = parts[1] || '';

		// Validate and trim decimals if needed
		if (!validateInput(cleaned)) {
			decimalPart = decimalPart.slice(0, MAX_DECIMALS[currency]);
		}

		// Add thousand separators
		integerPart = integerPart.replace(
			/\B(?=(\d{3})+(?!\d))/g,
			decimalSeparator === '.' ? ',' : '.'
		);

		return decimalPart
			? `${integerPart}${decimalSeparator}${decimalPart}`
			: integerPart;
	}

	function handleInput(event: Event) {
		const input = event.target as HTMLInputElement;
		const newValue = input.value;
		const caretPos = input.selectionStart || 0;
		const oldLength = newValue.length;

		const formatted = formatNumber(newValue);

		if (formatted !== value) {
			value = formatted;

			// Restore caret position
			setTimeout(() => {
				const newPos = caretPos + (formatted.length - oldLength);
				inputElement.setSelectionRange(newPos, newPos);
			}, 0);
		}
	}
</script>

// src/lib/components/CryptoInput.svelte
<div class="w-full max-w-md space-y-2">
	<div class="relative flex w-full">
		<input
			bind:this={inputElement}
			{value}
			on:input={handleInput}
			type="text"
			placeholder={`Enter amount (max ${MAX_DECIMALS[currency]} decimals)`}
			class="w-full px-4 py-2 text-lg border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
		/>
		<select
			bind:value={currency}
			class="px-3 py-2 text-lg font-medium text-gray-700 bg-gray-100 border border-l-0 rounded-r focus:outline-none focus:ring-2 focus:ring-blue-500"
		>
			<option value="ETH">ETH</option>
			<option value="BTC">BTC</option>
			<option value="USDT">USDT</option>
			<option value="IDR">IDR</option>
		</select>
	</div>

	{#if error}
		<p class="text-sm text-red-500">{error}</p>
	{/if}

	{#if value}
		<div class="p-3 bg-gray-50 rounded">
			<p class="text-sm text-gray-600">Current Value:</p>
			<p class="text-lg font-medium text-gray-800">{value} {currency}</p>
		</div>
	{/if}
</div>
