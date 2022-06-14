<div class="paginate-buttons">
	<button on:click={previous}> Previous </button>
	Page {page}
	<button on:click={next}> Next </button>
</div>
<div class="country-list">
	{#each worldData as countryData}
		<button class="countries" on:click={seeCities(countryData)}>{countryData.name}</button>
	{/each}
</div>
<div class="city-air-stats">
	<th>{currentCity}</th>
	{#each currentCityInfo as info}
		<tr>{info[0]} : {parseFloat(info[1]).toFixed(2)} {info[2]}</tr>
	{/each}
</div>
<div class="cities">
	<h2>{currentCountry}</h2>
	{#if cities.length > 0}
		{#each cities as city}
			{#if city.city != "N/A"}
				<button class="city-list-item" on:click={cityData(city)}>{city.city}</button>
			{/if}
		{/each}
	{/if}
</div>

<script>
import axios from 'axios';
let page = 1;
let count = 0;
let worldData = [];
let cities = [];
let cityInformation = [];
let currentCountry = '';
let currentCity = '';
let currentCityInfo = [];
let averageData = [];

const updateCountries = () => axios.get(`https://api.openaq.org/v2/countries?limit=20&page=${page}`)
	.then(response => worldData = response.data.results);

const seeCities = e => axios.get(`https://api.openaq.org/v2/cities?&country_id=${e.code}`)
	.then(response => {
		cities = (response.data.results);
		currentCity = '';
		currentCityInfo = [];
		currentCountry = e.name;
	});

const cityData = e =>
	axios.get(`https://api.openaq.org/v2/locations?city=${e.city}`)
		.then(response => {
			currentCityInfo = [];
			averageData = [];
			currentCity = e.city;
			cityInformation = response.data.results;
			cityInformation.forEach(element => 
				element.parameters.forEach(i =>
					currentCityInfo.push([i.displayName, i.average, i.unit])));
			currentCityInfo.sort();
			for (let i = 0; i <= currentCityInfo.length; i++) {
				let element = currentCityInfo;
				if (element[i + 1]) {
					if (element[i][0] === element[i + 1][0]) {
						count++;
						element[i + 1][1] += element[i][1];
						element[i + 1][3] = count;
					} else {
						count = 1;
						averageData.push(element[i]);
					}
				} else if (element[i]) averageData.push(element[i]);
			}
			currentCityInfo = [];
			averageData.forEach(e => {
				if (e.length === 3) e[3]= 1;
				e[1] /= e[3];
				currentCityInfo.push(e);
			});
		})

$: worldData, updateCountries();

const previous = () => {
	if (page > 1) return page --;
};

const next = () => {
	if (page < 8) return page ++;
};
</script>

<style>
.country-list {
	padding-top: 10rem;
	margin-left: auto;
    margin-right: auto;
	max-width: 50%;
}

.countries {
	cursor: pointer;
	padding: 2rem;
	margin: 2rem;
}

.cities {
	margin-left: auto;
    margin-right: auto;
	max-width: 50%;
}

.city-list-item {
	padding: 1rem;
	margin: 1rem;
}

.city-air-stats {
	position: absolute;
	margin-left: 10%;
}
</style>