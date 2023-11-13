<script>
	let offers = [
		{ price: 5, tokens: 3 },
		{ price: 6, tokens: 1 },
		{ price: 7, tokens: 1 },
		{ price: 5, tokens: 2 },
		{ price: 6, tokens: 3 }
	];
	let priceHistory = [
		{
			averagePrice: -1,
			averagePriceLastXTokens: -1,
			averagePriceLastXOffers: -1,
			weightedAveragePriceLastXOffers: -1
		},
		{
			averagePrice: -1,
			averagePriceLastXTokens: -1,
			averagePriceLastXOffers: -1,
			weightedAveragePriceLastXOffers: -1
		},
		{
			averagePrice: -1,
			averagePriceLastXTokens: -1,
			averagePriceLastXOffers: -1,
			weightedAveragePriceLastXOffers: -1
		},
		{
			averagePrice: -1,
			averagePriceLastXTokens: -1,
			averagePriceLastXOffers: -1,
			weightedAveragePriceLastXOffers: -1
		},
	];
	let newPrice = 0;
	let newTokenAmount = 0;
	let averagePrice = 0;

	let averagePriceLastXTokens = 0;
	const totalTokensToConsider = 20;

	let averagePriceLastXOffers = 0;
	const totalOffersToConsider = 5; // Change this to the number of last offers you want to consider

	let weightedAveragePriceLastXOffers = 0;
	const totalOffersToConsider2 = 5; // Change this to the number of last offers you want to consider

	$: totalTokens = offers.reduce((total, offer) => total + offer.tokens, 0);

	function addOffer() {
		if (newPrice > 0 && newTokenAmount > 0) {
			offers = [...offers, { price: newPrice, tokens: newTokenAmount }];
			calculateAveragePrice();
			calculateAveragePriceOfTokens(totalTokensToConsider);
			calculateAveragePriceOfLastXOffers(totalOffersToConsider);
			calculateWeightedAveragePriceOfLastXOffers(totalOffersToConsider2);
			updatePriceHistory();
			console.log(offers);
		}
	}

	function calculateAveragePrice() {
		let total = 0;
		let totalTokens = 0;
		for (let offer of offers) {
			total += offer.price * offer.tokens;
			totalTokens += offer.tokens;
		}
		averagePrice = totalTokens > 0 ? total / totalTokens : 0;
	}

	function calculateAveragePriceOfTokens(x) {
		let total = 0;
		let tokensCounted = 0;

		for (let i = offers.length - 1; i >= 0; i--) {
			const offer = offers[i];
			const tokensToTake = Math.min(offer.tokens, x - tokensCounted);

			total += offer.price * tokensToTake;
			tokensCounted += tokensToTake;

			if (tokensCounted >= x) break;
		}

		averagePriceLastXTokens = tokensCounted > 0 ? total / tokensCounted : 0;
	}

	function calculateAveragePriceOfLastXOffers(x) {
		let total = 0;
		let offersCounted = 0;

		for (let i = offers.length - 1; i >= 0 && offersCounted < x; i--) {
			total += offers[i].price;
			offersCounted++;
		}

		averagePriceLastXOffers = offersCounted > 0 ? total / offersCounted : 0;
	}
	function calculateWeightedAveragePriceOfLastXOffers(x) {
		let totalValue = 0;
		let totalTokens = 0;

		for (let i = offers.length - 1; i >= Math.max(offers.length - x, 0); i--) {
			totalValue += offers[i].price * offers[i].tokens;
			totalTokens += offers[i].tokens;
		}

		weightedAveragePriceLastXOffers = totalTokens > 0 ? totalValue / totalTokens : 0;
	}

	function updatePriceHistory() {
		calculateAveragePrice();
		calculateAveragePriceOfTokens(totalTokensToConsider);
		calculateAveragePriceOfLastXOffers(totalOffersToConsider);
		calculateWeightedAveragePriceOfLastXOffers(totalOffersToConsider);

		// Add the new set of calculations to the history
		priceHistory = [
			...priceHistory,
			{
				averagePrice: averagePrice,
				averagePriceLastXTokens: averagePriceLastXTokens,
				averagePriceLastXOffers: averagePriceLastXOffers,
				weightedAveragePriceLastXOffers: weightedAveragePriceLastXOffers
			}
		];
	}
	updatePriceHistory();
</script>

<div class="wrapper">
	<h1>Tradar-Price-Calculator</h1>
	<div class="container">
		<div>
			<h2>Add New Offer</h2>
			<input type="number" placeholder="Price" bind:value={newPrice} />
			<input type="number" placeholder="Token Amount" bind:value={newTokenAmount} />
			<button on:click={addOffer}>Add Offer</button>
		</div>
	</div>

	<div class="container">
		<div>
			<h2>Average Price</h2>
			<p>{averagePrice.toFixed(2)}</p>
		</div>

		<div>
			<h2>Average Price of Last {totalTokensToConsider} Tokens</h2>
			<p>{averagePriceLastXTokens.toFixed(2)}</p>
		</div>

		<div>
			<h2>Average Price of Last {totalOffersToConsider} Offers</h2>
			<p>{averagePriceLastXOffers.toFixed(2)}</p>
		</div>

		<div>
			<h2>Weighted Average Price of Last {totalOffersToConsider2} Offers</h2>
			<p>{weightedAveragePriceLastXOffers.toFixed(2)}</p>
		</div>
	</div>

	<div class="container">
		<div>
			<h2>Amount of offers {offers.length}</h2>
			<p>{offers.length}</p>
		</div>

		<div>
			<h2>Amount of tokens</h2>
			<p>{totalTokens}</p>
		</div>
	</div>

	<div class="container">
		<h2>Price History - newest first</h2>
		<div class="row">
			<table class="reference">
				<tr>
					<th>Ø Price</th>
					<th>Ø Last {totalTokensToConsider} Tokens</th>
					<th>Ø Last {totalOffersToConsider} Offers</th>
					<th>Ø Last {totalOffersToConsider2} weighted Offers</th>
				</tr>

				{#each [...priceHistory].reverse() as price}
					<tr>
						<td>{price.averagePrice.toFixed(2)}</td>
						<td>{price.averagePriceLastXTokens.toFixed(2)}</td>
						<td>{price.averagePriceLastXOffers.toFixed(2)}</td>
						<td>{price.weightedAveragePriceLastXOffers.toFixed(2)}</td>
					</tr>
				{/each}
			</table>
			<table class="reference">
				<tr>
					<th>Price</th>
					<th>Tokens</th>
				</tr>

				{#each [...offers].reverse() as offer}
					<tr>
						<td>{offer.price}</td>
						<td>{offer.tokens}</td>
					</tr>
				{/each}
			</table>
		</div>
	</div>
</div>

<style>
	th {
		text-align: start;
		padding-right: 20px;
		padding-bottom: 8px;
	}

	td {
		padding-bottom: 4px;
	}
	table.reference:nth-child(1) tr:nth-child(2) {
		background-color: rgb(223, 223, 223);
	}

	p,
	h1,
	h2,
	th,
	table {
		font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
			Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
	}

	h2 {
		font-size: 20px;
		font-weight: 500;
		color: rgb(37, 140, 40);
	}

	.wrapper {
		max-width: 920px;
		margin: 5%;
		gap: 10px;
	}

	.row {
		display: flex;
		flex-direction: row;
	}

	.container {
		border-radius: 8px;
		padding: 16px;
		background-color: rgb(242, 242, 242);
		margin-bottom: 20px;
	}
</style>
