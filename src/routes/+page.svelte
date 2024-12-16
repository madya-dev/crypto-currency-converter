<script>
	import { writable } from 'svelte/store';

	const preferredFormat = writable('comma'); // 'comma' for 1.000,00 or 'period' for 1,000.00
	let inputFromValue = '';
	let inputToValue = '';

	const decimals = {
		ETH: 18,
		BTC: 8,
		USDT: 6,
		IDR: 2,
	};

	let fromCurrency = 'USDT';
	let toCurrency = 'IDR';

	const exchangeRates = {
		IDR: { BTC: 0.000000004, ETH: 0.0000001, USDT: 0.00007 },
		BTC: { IDR: 250000000, ETH: 14.5, USDT: 30000 },
		ETH: { IDR: 17000000, BTC: 0.07, USDT: 2100 },
		USDT: { IDR: 14500, BTC: 0.000033, ETH: 0.00048 },
	};

	const getExchangeRate = (from, to) => {
		return exchangeRates[from]?.[to] || 1;
	};

	const autoConvert = () => {
		const rate = getExchangeRate(fromCurrency, toCurrency);
		if (inputFromValue) {
			// Remove formatting and convert to a number
			const normalizedValue = parseFloat(
				removeFormatting(inputFromValue, $preferredFormat)
			);
			inputToValue = (normalizedValue * rate).toFixed(decimals[toCurrency]);

			// Format the result back to the user preferred format
			inputToValue = formatNumber(inputToValue, $preferredFormat);
		} else {
			inputToValue = '';
		}
	};

	// Format a number string to the user preferred format
	const formatNumber = (value, format) => {
		const [integerPart, decimalPart] = value.split('.');

		if (format === 'comma') {
			// comma as decimal
			const formattedInteger = integerPart.replace(
				/\B(?=(\d{3})+(?!\d))/g,
				'.'
			);
			return decimalPart
				? `${formattedInteger},${decimalPart}`
				: formattedInteger;
		} else {
			// period as decimal
			const formattedInteger = integerPart.replace(
				/\B(?=(\d{3})+(?!\d))/g,
				','
			);
			return decimalPart
				? `${formattedInteger}.${decimalPart}`
				: formattedInteger;
		}
	};

	// Remove formatting to parse the number
	const removeFormatting = (value, format) => {
		if (format === 'comma') {
			// Remove thousands '.' and replace ',' with '.'
			return value.replace(/\./g, '').replace(',', '.');
		} else {
			// Remove thousands ',' (Period format uses '.' as the decimal separator)
			return value.replace(/,/g, '');
		}
	};

	// Validate the decimal precision based on the selected currency
	const validateDecimals = (value) => {
		const [integerPart, decimalPart] = value.split(
			preferredFormat === 'comma' ? ',' : '.'
		);
		const allowedDecimals = decimals[fromCurrency];

		// If the decimal part exists and exceeds the allowed precision, reject the input
		if (decimalPart && decimalPart.length > allowedDecimals) {
			return false;
		}

		return true;
	};

	const onFromInputChange = (event) => {
		inputFromValue = event.target.value;

		// Allow only valid input characters (number, ',', '.')
		const validInputRegex =
			$preferredFormat === 'comma' ? /^[\d.,]*$/ : /^[\d.,]*$/;

		// Check if the input has exceeded the decimal precision
		if (validInputRegex.test(inputFromValue)) {
			// Check for decimals and prevent further input if the limit is exceeded
			const [integerPart, decimalPart] = inputFromValue.split(
				preferredFormat === 'comma' ? ',' : '.'
			);
			const allowedDecimals = decimals[fromCurrency];

			// If the decimal part exceeds the allowed precision, limit further input
			if (decimalPart && decimalPart.length > allowedDecimals) {
				inputFromValue = inputFromValue.slice(0, inputFromValue.length - 1); // Remove last character
			}

			autoConvert();
		}
	};
</script>

<div class="mx-auto p-8 text-white min-h-screen bg-gray-900">
	<div
		class="max-w-xl mx-auto border border-gray-700 rounded-2xl bg-gray-900 shadow-lg"
	>
		<h1 class="text-3xl font-bold text-center py-4">Instant Convert</h1>

		<div class="p-6">
			<div class="mb-6">
				<label class="block text-sm font-medium mb-2">Preferred Format</label>
				<select
					bind:value={$preferredFormat}
					class="bg-transparent text-white focus:outline-none p-2 rounded-lg"
				>
					<option value="comma">Comma as Decimal (1.000,00)</option>
					<option value="period">Period as Decimal (1,000.00)</option>
				</select>
			</div>

			<div class="mb-6">
				<label class="block text-sm font-medium mb-2">From</label>
				<div class="flex justify-between items-center rounded-lg">
					<select
						bind:value={fromCurrency}
						on:change={autoConvert}
						class="text-white focus:outline-none bg-gray-600 p-2 rounded-lg"
					>
						<option value="IDR">IDR</option>
						<option value="BTC">BTC</option>
						<option value="ETH">ETH</option>
						<option value="USDT">USDT</option>
					</select>
				</div>
				<input
					type="text"
					bind:value={inputFromValue}
					on:input={onFromInputChange}
					placeholder="0"
					class="w-full mt-3 p-3 rounded-md bg-gray-700 text-white focus:ring focus:ring-green-400 focus:outline-none"
				/>
			</div>
			<hr class="my-6 border-gray-600" />
			<div class="mb-6">
				<label class="block text-sm font-medium mb-2">To</label>
				<div class="flex justify-between items-centerrounded-lg">
					<select
						bind:value={toCurrency}
						on:change={autoConvert}
						class="text-white focus:outline-none bg-gray-600 p-2 rounded-lg"
					>
						<option value="IDR">IDR</option>
						<option value="BTC">BTC</option>
						<option value="ETH">ETH</option>
						<option value="USDT">USDT</option>
					</select>
				</div>
				<input
					type="text"
					bind:value={inputToValue}
					disabled
					placeholder="0"
					class="w-full mt-3 p-3 rounded-md bg-gray-700 text-white focus:outline-none"
				/>
			</div>
		</div>
	</div>
</div>

<style></style>
