<div class="landing">
	<h2>Air Quality App</h2>
	<h3>Search Location with Latidtude and Longitude Co-Ordinates</h3>
</div>
<div class="search-form">
	<input type="number" placeholder="Latitude" bind:value={ordinates.lat}/>
	<input type="number" placeholder="Longitude" bind:value={ordinates.long}/>
	<button on:click={search}>Search Nearest Location</button> 
</div>
<div class="search-result">
	{#if searchResultLocation}
		<th>{searchResultLocation} loacted in {searchResultCity} : {searchResultCountry}</th>
		{#each citySearchInfo as info}
			<tr>{info[0]} : {parseFloat(info[1]).toFixed(2)} {info[2]}</tr>
		{/each}
		{:else if error}
			<div class="warning-message">
				The co-ordinates you entered didn't have anything to return,
				please try again.
			</div>
	{/if}
</div>

<script>
import axios from 'axios';

let ordinates = {
	lat: null,
	long: null,
};
let citySearchInfo = [];
let searchResults = [];
let searchResultLocation = '';
let searchResultCountry = '';
let searchResultCity = '';
let error = false;

const search = () => {
	error = false;
	axios.get(`https://api.openaq.org/v2/locations?limit=100&coordinates=${ordinates.lat}%2C${ordinates.long}&radius=100000`).then
		((response) => {
			citySearchInfo = [];
			searchResultLocation = '';
			searchResultCountry = '';
			if (!response.data.results.length) error = true;
			else {
				let dataSet = response.data.results
				let nearestIndex = null;
				let nearestDistance = 100000;
				dataSet.forEach((element, index) => {
					if (element.coordinates) {
						let distance = Math.sqrt(
							Math.pow(element.coordinates.latitude + ordinates.lat,2) +
							Math.pow(element.coordinates.longitude + ordinates.long,2)
						);
						if (distance < nearestDistance) {
							nearestIndex = index;
							nearestDistance = distance;
						}
					}
				});
				let data = response.data.results[nearestIndex];
				searchResultLocation = data.name;
				searchResultCity = data.city;
				searchResultCountry = data.country;
				searchResults = data.parameters;
				searchResults.forEach(element => citySearchInfo.push([element.displayName, element.average, element.unit]));
				ordinates.lat = null;
				ordinates.long = null;
			}
		});
};
</script>

<style>
.landing, .search-form{
	text-align: center;
}
.search-result {
	position: absolute;
	margin-left: 50%;
	transform: translateX(-50%);
	margin-top: 5rem;
}

.warning-message {
	color: red;
}
</style>