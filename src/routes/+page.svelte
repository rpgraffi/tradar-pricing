<script>
	import Chart from '../components/chart.svelte';
	let offers = [
		{ price: 5, tokens: 3 },
		{ price: 6, tokens: 1 },
		{ price: 7, tokens: 1 },
		{ price: 5, tokens: 2 },
		{ price: 6, tokens: 3 }
	];
	let priceHistory = [
		{
			averagePrice: 0,
			averagePriceLastXTokens: 0,
			averagePriceLastXOffers: 0,
			weightedAveragePriceLastXOffers: 0
		},
		{
			averagePrice: 0,
			averagePriceLastXTokens: 0,
			averagePriceLastXOffers: 0,
			weightedAveragePriceLastXOffers: 0
		},
		{
			averagePrice: 0,
			averagePriceLastXTokens: 0,
			averagePriceLastXOffers: 0,
			weightedAveragePriceLastXOffers: 0
		},
		{
			averagePrice: 0,
			averagePriceLastXTokens: 0,
			averagePriceLastXOffers: 0,
			weightedAveragePriceLastXOffers: 0
		}
	];
	let newPrice = 0;
	let newTokenAmount = 0;
	let averagePrice = 0;

	let averagePriceLastXTokens = 0;
	$: totalTokensToConsider = 20;

	let averagePriceLastXOffers = 0;
	$: totalOffersToConsider = 5; // Change this to the number of last offers you want to consider

	let weightedAveragePriceLastXOffers = 0;
	$: totalOffersToConsider2 = 5; // Change this to the number of last offers you want to consider

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
			<h2>{offers.length} Offers & {totalTokens} Tokens.</h2>
		</div>
	</div>
	<div class="container">
		<div>
			<h2>Add New Offer</h2>
            <div class="row">
                <div class="col">
                    <label for="Price">Price</label>
                    <input type="number" placeholder="eg. 10" bind:value={newPrice} />
                </div>
                <div class="col">
                    <label for="Tokens">Tokens</label>
                    <input type="number" placeholder="eg. 3" bind:value={newTokenAmount} />
                </div>
                <div class="col">
                    <label for="tokensToConsider">Last {totalTokensToConsider} tokens</label>
                    <input type="number" placeholder="eg. 10" bind:value={totalTokensToConsider} />
                </div>
                <div class="col">
                    <label for="tokensToConsider">Last {totalOffersToConsider} tokens</label>
                    <input type="number" placeholder="eg. 10" bind:value={totalOffersToConsider} />
                </div>
                <div class="col">
                    <label for="tokensToConsider">Last {totalOffersToConsider2} weighted tokens</label>
                    <input type="number" placeholder="eg. 10" bind:value={totalOffersToConsider2} />
                </div>

            </div>
			<button on:click={addOffer}>Add Offer</button>
		</div>
	</div>


    <div class="container">
		<Chart data={priceHistory} />
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
						<td class="price">{price.averagePriceLastXOffers.toFixed(2)}</td>
						<td class="price">{price.weightedAveragePriceLastXOffers.toFixed(2)}</td>
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
						<td class="price">{offer.price} €</td>
						<td class="price">{offer.tokens}</td>
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
		color: black;
	}

	td {
		padding-bottom: 4px;
	}
	table.reference:nth-child(1) tr:nth-child(2) {
		background-color: rgb(223, 223, 223);
	}

	tr:nth-child(-n + 6) .price {
		color: rgb(147, 0, 200);
	}

	p,
	h1,
	h2,
	th,
	table,
	label {
		font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
			Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
	}

	a {
		text-decoration: none;
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
	.col {
		display: flex;
		flex-direction: column;
	}

	.container {
		border-radius: 8px;
		padding: 16px;
		background-color: rgb(242, 242, 242);
		margin-bottom: 20px;
	}
</style>
