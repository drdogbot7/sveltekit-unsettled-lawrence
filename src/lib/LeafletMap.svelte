<script>
	import { onMount, onDestroy } from 'svelte';

	let mapElement;
	let map;

	onMount(async () => {
		const L = await import('leaflet');

		map = L.map(mapElement, {
			center: [38.97, -95.23],
			zoom: 15,
			minZoom: 15,
			maxZoom: 19
		});

		// Add icon colors
		const iconCommon = {
			shadowUrl: 'img/marker-shadow.png',
			iconSize: [25, 41],
			iconAnchor: [12, 41],
			popupAnchor: [1, -34],
			shadowSize: [41, 41]
		};
		const iconGreen = new L.Icon({
			iconUrl: 'img/marker-icon-green.png',
			iconRetinaUrl: 'img/marker-icon-2x-green.png',
			...iconCommon
		});
		const iconYellow = new L.Icon({
			iconUrl: 'img/marker-icon-yellow.png',
			iconRetinaUrl: 'img/marker-icon-2x-yellow.png',
			...iconCommon
		});
		const iconRed = new L.Icon({
			iconUrl: 'img/marker-icon-red.png',
			iconRetinaUrl: 'img/marker-icon-2x-red.png',
			...iconCommon
		});
		const iconOrange = new L.Icon({
			iconUrl: 'img/marker-icon-orange.png',
			iconRetinaUrl: 'img/marker-icon-2x-orange.png',
			...iconCommon
		});
		const iconBlue = new L.Icon({
			iconUrl: 'img/marker-icon-blue.png',
			iconRetinaUrl: 'img/marker-icon-2x-blue.png',
			...iconCommon
		});

		// Define basemap
		const positron = L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
			attribution:
				'&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/">CARTO</a>',
			subdomains: 'abcd',
			maxZoom: 19
		});
		positron.addTo(map);

		// Create Sanborn Map overlay layers

		const overlayCommon = {
			tms: 1,
			opacity: 1,
			minZoom: 15,
			maxZoom: 19
		};
		const overlay1883 = L.tileLayer('tiles/1883/{z}/{x}/{y}.webp', {
			attribution: '1883 Sanborn Map',
			...overlayCommon
		});
		const overlay1887 = L.tileLayer('tiles/1887/{z}/{x}/{y}.webp', {
			attribution: '1887 Sanborn Map',
			...overlayCommon
		}).addTo(map); // Default to 1887 map
		const overlay1889 = L.tileLayer('tiles/1889/{z}/{x}/{y}.webp', {
			attribution: '1889 Sanborn Map',
			...overlayCommon
		});
		const overlay1905 = L.tileLayer('tiles/1905/{z}/{x}/{y}.webp', {
			attribution: '1905 Sanborn Map',
			...overlayCommon
		});

		// Add historic map overlays to a layer control
		const overlaymaps = {
			'1883 Map': overlay1883,
			'1887 Map': overlay1887,
			'1889 Map': overlay1889,
			'1905 Map': overlay1905
		};

		const layerControl = L.control.layers(null, overlaymaps, { collapsed: false }).addTo(map);

		// Create a feature group for the data points
		const dataOverlay = L.featureGroup();
		const markers = [];

		// Fetch data from data.json
		fetch('/data.json')
			.then((response) => response.json())
			.then((data) => {
				data.forEach((item) => {
					const popupContent = `
            <strong>Newspaper:</strong> ${item.Newspaper || 'N/A'}<br>
            <strong>Date:</strong> ${item.Date || 'N/A'}<br>
            <strong>Location:</strong> ${item.LocationTitle || 'N/A'}<br>
            <strong>Location Keywords:</strong> ${item.KeywordLocation || 'N/A'}<br>
            <strong>Affiliations:</strong> ${item.Affiliations || 'N/A'}<br>
            <strong>Language:</strong> ${item.Language || 'N/A'}
          `;
					// Determine icon color based on location
					// Default to blue if no location matches
					let myIcon;
					switch (item.LocationTitle) {
						case 'East Bottoms':
							myIcon = iconGreen;
							break;
						case 'Poverty Flat':
							myIcon = iconYellow;
							break;
						case 'Smokey Pilgrims':
							myIcon = iconOrange;
							break;
						case 'Bad Lands':
							myIcon = iconRed;
							break;
						default:
							myIcon = iconBlue; // Default icon
					}
					const marker = L.marker([item.Latitude, item.Longitude], {
						icon: myIcon
					}).bindPopup(popupContent);
					marker.year = parseInt(item.Date.split('/').pop(), 10); // Extract year from the date
					marker.location = item.LocationTitle;
					markers.push(marker);
					dataOverlay.addLayer(marker);
				});

				dataOverlay.addTo(map);
			})
			.catch((error) => console.error('Error loading data:', error));

		// Add filter controls
		const filterContainer = L.DomUtil.create('div', 'filter-container');
		filterContainer.innerHTML = `
  <div class="filter filter--years">
    <label for="min-year-slider">Filter by Year:</label>
    <div class="year-slider">
      <span>Min Year:</span>
      <input type="range" id="min-year-slider" min="1880" max="1925" step="1" value="1880">
      <span id="min-year-value" class="year-slider__value">1880</span>
    </div>
    <div class="year-slider">
      <span>Max Year:</span>
      <input type="range" id="max-year-slider" min="1880" max="1925" step="1" value="1925">
      <span id="max-year-value" class="year-slider__value">1925</span>
    </div>
  </div>
  <div class="filter filter--locations">
    <label for="location-filter">Filter by Location:</label>
    <select id="location-filter">
      <option value="all">All</option>
      <option value="East Bottoms">East Bottoms</option>
      <option value="Poverty Flat">Poverty Flat</option>
      <option value="Smokey Pilgrims">Smokey Pilgrims</option>
      <option value="Bad Lands">Bad Lands</option>
    </select>
  </div>
`;

		const minYearSlider = filterContainer.querySelector('#min-year-slider');
		const maxYearSlider = filterContainer.querySelector('#max-year-slider');
		const minYearValue = filterContainer.querySelector('#min-year-value');
		const maxYearValue = filterContainer.querySelector('#max-year-value');
		const locationSelector = filterContainer.querySelector('#location-filter');

		const updateMarkers = () => {
			const minYear = parseInt(minYearSlider.value, 10);
			const maxYear = parseInt(maxYearSlider.value, 10);
			const selectedLocation = locationSelector.value;

			minYearValue.textContent = minYear;
			maxYearValue.textContent = maxYear;

			dataOverlay.clearLayers(); // Clear all markers

			markers.forEach((marker) => {
				if (
					marker.year >= minYear &&
					marker.year <= maxYear &&
					(selectedLocation == marker.location || selectedLocation == 'all')
				) {
					dataOverlay.addLayer(marker); // Add marker if it matches the filters
				}
			});
		};

		minYearSlider.addEventListener('input', updateMarkers);
		maxYearSlider.addEventListener('input', updateMarkers);
		locationSelector.addEventListener('change', updateMarkers);

		// Add the filters to the layer control
		const layerControlContainer = layerControl.getContainer();
		L.DomEvent.disableClickPropagation(filterContainer);
		layerControlContainer.appendChild(filterContainer);

		const legend = `
<div class="legend">
      <div class="legend__item">
        <div class="legend__icon">
          <img srcset="/img/marker-icon-green.png, /img/marker-icon-2x-green.png 2x" src="/img/marker-icon-green.png" alt="Green Map Marker" />
        </div>
        <div class="legend__label">East Bottoms</div>
      </div>
      <div class="legend__item">
        <div class="legend__icon">
          <img srcset="/img/marker-icon-yellow.png, /img/marker-icon-2x-yellow.png 2x" src="/img/marker-icon-yellow.png" alt="Yellow Map Marker" />
        </div>
        <div class="legend__label">Poverty Flat</div>
      </div>
      <div class="legend__item">
        <div class="legend__icon">
          <img srcset="/img/marker-icon-orange.png, /img/marker-icon-2x-orange.png 2x" src="/img/marker-icon-orange.png" alt="Orange Map Marker" />
        </div>
        <div class="legend__label">Smokey Pilgrims</div>
      </div>
      <div class="legend__item">
        <div class="legend__icon">
          <img srcset="/img/marker-icon-red.png, /img/marker-icon-2x-red.png 2x" src="/img/marker-icon-red.png" alt="Red Map Marker" />
        </div>
        <div class="legend__label">Bad Lands</div>
        </div>
      <div class="legend__item">
        <div class="legend__icon">
          <img srcset="/img/marker-icon-blue.png, /img/marker-icon-2x-blue.png 2x" src="/img/marker-icon-blue.png" alt="Blue Map Marker" />
        </div>
        <div class="legend__label">Other</div>
      </div
    </div>
  </div>
  `;
		const legendContainer = L.control({ position: 'bottomright' });
		legendContainer.onAdd = function () {
			const div = L.DomUtil.create(
				'div',
				'leaflet-control-layers leaflet-control-layers-expanded leaflet-control'
			);
			div.innerHTML = legend;
			return div;
		};
		legendContainer.addTo(map);
	});

	onDestroy(async () => {
		if (map) {
			console.log('Unloading Leaflet map.');
			map.remove();
		}
	});
</script>

<main>
	<div bind:this={mapElement}></div>
</main>

<style>
	@import 'leaflet/dist/leaflet.css';
	main div {
		height: calc(100vh - 44px);
	}
</style>
