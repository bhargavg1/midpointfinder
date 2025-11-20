
<script lang='ts'>
    import { onMount } from "svelte";
    import { PUBLIC_GOOGLE_MAPS_API_KEY } from '$env/static/public';

    let mapelement: any;
    let map: any;

    let autocomplete: any;

    let markers: any = [];

    let herecoords: any = [];
    $: console.log("the coords are " + herecoords);

    onMount(() => {

        if (typeof window !== 'undefined') {            
            const script = document.createElement('script');
            script.src = `https://maps.googleapis.com/maps/api/js?libraries=places,visualization,marker&key=${PUBLIC_GOOGLE_MAPS_API_KEY}&v=weekly`;
            script.async = true;
            script.defer = true;

            script.onload = async () => {
                const { Map } = (await google.maps.importLibrary("maps")) as google.maps.MapsLibrary;
                map = new Map(mapelement, {
                    center: { lat: 33.956386, lng: -84.381568},
                    zoom: 4,
                    mapId: "9cee6a35719fe30a71f4b407",

                    mapTypeControlOptions: {
                        style: google.maps.MapTypeControlStyle.DEFAULT,
                        position: google.maps.ControlPosition.RIGHT_TOP
                    },

                    fullscreenControl: false,
                    streetViewControl: false,
                });
            }

            document.head.appendChild(script);
        }


    })

    let yoursuggestions: any = [];

    let timeout: any;
    const onyourplacechange = function (e: any) {
        clearTimeout(timeout);

        yoursuggestions = [];

        timeout = setTimeout(async () => {
            console.log(e.target.value);
            
            if (e.target.value == "") {
                return;
            }

            const sessiontoken = new google.maps.places.AutocompleteSessionToken();
            const { suggestions } = await google.maps.places.AutocompleteSuggestion.fetchAutocompleteSuggestions({
                sessionToken: sessiontoken,
                input: e.target.value
            });

            const service = new google.maps.places.PlacesService(map);
            suggestions.forEach((x) => {
                //console.log("prediction: ", x.placePrediction?.placeId);
                let result: string = "";
                if (x.placePrediction) {
                    service.getDetails({ placeId: x.placePrediction?.placeId}, (place, status) => {
                        if (status === google.maps.places.PlacesServiceStatus.OK) {
                            console.log(place?.name, " Located at: ", place?.formatted_address);
                            yoursuggestions = [...yoursuggestions, 
                            {
                                text: (place?.name +  " Located at: " +  place?.formatted_address),
                                id: place?.place_id
                            }];
                        }
                    })
                }
            });
        }, 600);
    }

    let youplaceid: any;
    let yourmapmarker: any;

    const onpressyou = async function (input: string) {
        const container = document.createElement("div");
        container.style.display = "flex";
        container.style.flexDirection = "column";
        container.style.alignItems = "center";
        const icon = document.createElement("img");
        icon.src = "https://maps.google.com/mapfiles/ms/icons/red-dot.png";
        icon.style.width = "40px";
        icon.style.height = "40px";
        const label = document.createElement("div");
        label.textContent = "Your Place";
        label.style.background = "white";
        label.style.border = "1px solid black";
        label.style.padding = "4px 8px";
        label.style.fontSize = "20px";
        label.style.fontWeight = "bold";
        label.style.borderRadius = "4px";
        container.appendChild(label);
        container.appendChild(icon);

        const place = new google.maps.places.Place({id: input});
        youplaceid = input;
        await place.fetchFields({fields: ["location", "formattedAddress"]});

        if (yourmapmarker) {
            yourmapmarker.map = null;
        }

        yourmapmarker = new google.maps.marker.AdvancedMarkerElement({
            position: place.location,
            map: map,
            title: "Your Place",
            content: container
        });

        const inputbox: HTMLInputElement = document.getElementById("yourloc") as HTMLInputElement;
        if (inputbox && place.formattedAddress) {
            inputbox.value = place.formattedAddress;
        }
    }

    let themplaceid: any;
    let themmapmarker: any;

    const onpressthem = async function (input: string) {
        const container = document.createElement("div");
        container.style.display = "flex";
        container.style.flexDirection = "column";
        container.style.alignItems = "center";
        const icon = document.createElement("img");
        icon.src = "https://maps.google.com/mapfiles/ms/icons/blue-dot.png";
        icon.style.width = "40px";
        icon.style.height = "40px";
        const label = document.createElement("div");
        label.textContent = "Their Place";
        label.style.background = "white";
        label.style.border = "1px solid black";
        label.style.padding = "4px 8px";
        label.style.fontSize = "20px";
        label.style.fontWeight = "bold";
        label.style.borderRadius = "4px";
        container.appendChild(label);
        container.appendChild(icon);

        const place = new google.maps.places.Place({id: input});
        themplaceid = input;
        await place.fetchFields({fields: ["location", "formattedAddress"]});

        if (themmapmarker) {
            themmapmarker.map = null;
        }
        themmapmarker = new google.maps.marker.AdvancedMarkerElement({
            position: place.location,
            map: map,
            title: "Their Place",
            content: container
        });

        const inputbox = document.getElementById("themloc");
        if (inputbox && place.formattedAddress) {
            inputbox.value = place.formattedAddress;
        }
    }

    

    let theirsuggestions: any = [];

    let timeout2: any;
    const ontheirplacechange = function (e: any) {

        clearTimeout(timeout2);

        theirsuggestions = [];

        timeout2 = setTimeout(async () => {
            console.log(e.target.value);

            if (e.target.value == "") {
                return;
            }

            const sessiontoken = new google.maps.places.AutocompleteSessionToken();
            const { suggestions } = await google.maps.places.AutocompleteSuggestion.fetchAutocompleteSuggestions({
                sessionToken: sessiontoken,
                input: e.target.value
            });

            const service = new google.maps.places.PlacesService(map);
            suggestions.forEach((x) => {
                //console.log("prediction: ", x.placePrediction?.placeId);
                let result: string = "";
                if (x.placePrediction) {
                    service.getDetails({ placeId: x.placePrediction?.placeId}, (place, status) => {
                        if (status === google.maps.places.PlacesServiceStatus.OK) {
                            console.log(place?.name, " Located at: ", place?.formatted_address);
                            theirsuggestions = [...theirsuggestions, 
                            {
                                text: (place?.name +  " Located at: " +  place?.formatted_address),
                                id: place?.place_id
                            }];                 
                        }
                    })
                }
            });
            

        }, 300);
    }

    const onrequestpress = function() {
        console.log("button pressed, ", herecoords);
        const buttoninner = document.getElementById("getyourlocation")?.childNodes[0];
        const buttoninnerfront = document.getElementById("getyourlocationfront")?.childNodes[0];
        if (buttoninner && buttoninnerfront) {
            buttoninner.textContent = "Loading...";
            buttoninnerfront.textContent = "🚫";
        }
        if ("geolocation" in navigator) {
            navigator.geolocation.getCurrentPosition(
                (pos) => {
                    herecoords = [pos.coords.latitude, pos.coords.longitude];
                    map.setCenter({
                        lat: herecoords[0],
                        lng: herecoords[1]
                    });
                    map.zoom = 12;
                    if (buttoninner && buttoninnerfront) {
                        buttoninner.textContent = "Request Location";
                        buttoninnerfront.textContent = "📍";
                    }

                    const container = document.createElement("div");
                    container.style.display = "flex";
                    container.style.flexDirection = "column";
                    container.style.alignItems = "center";
                    const icon = document.createElement("img");
                    icon.src = "https://maps.google.com/mapfiles/ms/icons/red-dot.png";
                    icon.style.width = "40px";
                    icon.style.height = "40px";
                    const label = document.createElement("div");
                    label.textContent = "Your Place";
                    label.style.background = "white";
                    label.style.border = "1px solid black";
                    label.style.padding = "4px 8px";
                    label.style.fontSize = "20px";
                    label.style.fontWeight = "bold";
                    label.style.borderRadius = "4px";
                    container.appendChild(label);
                    container.appendChild(icon);

                    if (yourmapmarker) {
                        yourmapmarker.map = null;
                    }
                    yourmapmarker = new google.maps.marker.AdvancedMarkerElement({
                        position: {
                            lat: herecoords[0],
                            lng: herecoords[1]
                        },
                        map: map,
                        title: "Your Place",
                        content: container
                    });

                    new google.maps.Geocoder().geocode(
                        {
                            location: {
                            lat: parseFloat(herecoords[0]),
                            lng: parseFloat(herecoords[1])
                        }},
                        (results, status) => {
                            if (status == "OK") {
                                const inputbox = document.getElementById("yourloc");
                                if (results && results[0] && inputbox) {
                                    inputbox.value = results[0].formatted_address;
                                    youplaceid = results[0].place_id;
                                } else {
                                    console.log("no address");
                                }
                            } else {
                                console.log("error geocoding");
                            }
                        }
                    );


                },
                (err) => {
                    console.error("there was an error: ", err);
                }
            );
        } else {
            console.error("cannot get the location");
        }
    }

    let submitbuttontext = "➡️";
    let directionresult: google.maps.DirectionsLeg;
    let renderer: any;

    const getDirections = function () {
        submitbuttontext = "Loading🔄️"
        const route = new google.maps.DirectionsService();
        if (renderer) {
            renderer.setMap(null);
        }
        renderer = new google.maps.DirectionsRenderer({map});
        route.route(
            {
                origin: { placeId: youplaceid },
                destination: { placeId: themplaceid },
                travelMode: google.maps.TravelMode.DRIVING
            },
            (result, status) => {
                submitbuttontext = "➡️";
                if (status === "OK") {
                    renderer.setDirections(result);
                    if (result) {
                        directionresult = result?.routes[0].legs[0];
                    }
                } else {
                    console.log("there was an error");
                }
            }
        )
    }


    let middleplaceid: any;
    let middlelatlng: any;
    let middlemapmarker: any;

    const getplaceid = function (latlng: google.maps.LatLng, x: (arg0: any) => void) {
        new google.maps.Geocoder().geocode(
            {
                location: {
                lat: parseFloat(herecoords[0]),
                lng: parseFloat(herecoords[1])
            }},
            (results, status) => {
                if (status == "OK") {
                    const inputbox = document.getElementById("yourloc");
                    if (results && results[0] && inputbox) {
                        x(results[0].place_id);
                    } else {
                        console.log("no address");
                    }
                } else {
                    console.log("error geocoding");
                }
            }
        );
    }

    const getMiddle = function () {

        const container = document.createElement("div");
        container.style.display = "flex";
        container.style.flexDirection = "column";
        container.style.alignItems = "center";
        const icon = document.createElement("img");
        icon.src = "https://maps.google.com/mapfiles/ms/icons/green-dot.png";
        icon.style.width = "40px";
        icon.style.height = "40px";
        const label = document.createElement("div");
        label.textContent = "Midpoint";
        label.style.background = "white";
        label.style.border = "1px solid black";
        label.style.padding = "4px 8px";
        label.style.fontSize = "20px";
        label.style.fontWeight = "bold";
        label.style.borderRadius = "4px";
        container.appendChild(label);
        container.appendChild(icon);

        const steps = directionresult.steps;
        const totaltime = steps.reduce((sum, step) => {
            if (step.duration) {
                return sum + step.duration?.value;   
            } else {
                return 0;
            }
        }, 0);
        const halftime = totaltime / 2;

        let elapsed = 0;
        let latlng: google.maps.LatLng;

        for (const step of steps) {
            const stepduration = step.duration!.value!;
            if (elapsed + stepduration >= halftime) {
                const remaining = halftime - elapsed;
                const fraction = remaining / stepduration;

                const encoded = step.polyline!.points!;
                if (!encoded) {
                    latlng = step.start_location;
                    middlelatlng = latlng;
                    getplaceid(latlng, (x) => {middleplaceid = x});      
                    if (middlemapmarker) {
                        middlemapmarker.map = null;
                    }
                    middlemapmarker = new google.maps.marker.AdvancedMarkerElement({
                        position: latlng,
                        map: map,
                        title: "Their Place",
                        content: container
                    });
                }

                const path = google.maps.geometry.encoding.decodePath(encoded);

                const index = Math.floor(fraction * (path.length - 1));
                latlng = path[index];
                middlelatlng = latlng;

                if (middlemapmarker) {
                    middlemapmarker.map = null;
                }
                middlemapmarker = new google.maps.marker.AdvancedMarkerElement({
                    position: latlng,
                    map: map,
                    title: "The middle",
                    content: container
                });
                break;
            }

            elapsed += stepduration;
        }
    }

    let midpointsuggestions: any = [];

    let timeout3: any;

    const onmidchange = function (e: any) {

        clearTimeout(timeout3);

        midpointsuggestions = [];

        timeout3 = setTimeout(async () => {
            console.log(e.target.value);

            if (e.target.value == "") {
                return;
            }

            const service = new google.maps.places.PlacesService(map);
            const request: google.maps.places.TextSearchRequest = {
                location: middlelatlng,
                radius: 5000,
                query: e.target.value
            }

            service.textSearch(request, (results, status) => {
                if (status === google.maps.places.PlacesServiceStatus.OK && results) {
                    results.forEach((place) => {
                        const photourls = place.photos;
                        midpointsuggestions = [...midpointsuggestions, {
                            name: place.name,
                            address: place.formatted_address,
                            id: place.place_id,
                            photos: place.photos![0].getUrl(),
                        }]
                    });
                }
                console.log("the locations: ", midpointsuggestions);
            });
            

        }, 300);
    }

</script>

<style>
    .inputdiv {
        background-color: rgba(95, 158, 160, 0.5);
        border-radius: 5px;
        box-shadow: 10px 5px 5px rgba(47, 48, 48, 0.5);  
        padding: 10px;
        width: 80%;
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        pointer-events: auto;
    }

    .inputholder {
        width: 100%;
        padding: 10px;
        pointer-events: auto;
        display: flex;
        justify-content: flex-start;
        align-items: center;
        pointer-events: none;
    }

    .inputbox {
        background-color:azure;
        border-radius: 5px;
        padding: 4px;
        width: 70%;
        transition: 200ms;
        border-width: 4px;
    }
    
    .inputboxfull {
        background-color: azure;
        border-radius: 5px;
        padding: 4px;
        width: 100%;
        transition: 200ms;
        border-width: 4px;
    }

    .inputbox:hover, .inputboxfull:hover {
        width: 100%;
        transition: 200ms;
    }

    .inputbox:focus, .inputboxfull:focus {
        width: 100%;
        background-color: lightgray;
    }

    .inputwrapper {
        padding: 10px;
        width: 100%;
    }

    .submitbutton {
        background-color:cornflowerblue;
        border-radius: 10px;
        padding: 10px;
        width: 100%;
    }

    .submitbutton:hover {
        background-color:darkseagreen;
        transition: 200ms;
    }

    .inputitem {
        width: 100%;
        display: flex;
        justify-content: flex-start;
        height: 3em;
    }

    .labels {
        font-size: 1.4em;
        
        padding: 10px;
    }
    #map {
        width: 100%;
        height: 100%;
        position: absolute;
        padding: 10%;
    }

    .overlay {
        position: absolute;
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
        justify-content: space-between;
        align-items: flex-start;
        width: 100%;
        height: 100%;
        pointer-events: none;
    }

    .font {
        font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
    }

    .logobox {
        display: flex;
        justify-content: space-between; 
        align-items: center;
        flex-direction: column;
        width: 100%;
        height: 100%;
        padding: 0.2em;
    }

    .logosurround {
        font-size: 3em;
        font-weight: 400;
        background-color: rgba(95, 158, 160, 0.5);
        box-shadow: 10px 5px 5px rgba(47, 48, 48, 0.5);
        padding: 0.5em;
        border-radius: 0.2em;
    }

    .inputcar {
        font-size: 3em;
    }

    .getbuttonbox {
        padding-left: 0.5em;
        position: relative;
        transition: 200ms;
        flex: 1;
        transition: 200ms;
        span {
            transition: 500ms;
        }
    }

    .getbuttonfront {
        background-color:azure;
        border-radius: 0.2em;
        width: 93%;
        font-size: 2em;
        transition: 200ms;
        backface-visibility: hidden;
        position: absolute;
    }

    .getbuttonback {
        background-color: cadetblue;
        border-radius: 0.2em;
        width: 93%;
        font-size: 2em;
        transition: 200ms;
        backface-visibility: hidden;
        position: absolute;
        transform: rotateY(180deg);
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100%;
        span {
            font-size: 0.5em;
        }
    }

    .getbuttonbox:hover  {
        .getbuttonfront {
            transition: 200ms;
            transform: rotateY(180deg);
        }

        .getbuttonback {
            transition: 200ms;
            transform: rotateY(0deg);            
        }
    }

    .inputitem .inputbox:hover ~ .getbuttonbox {
        transform: scaleX(0);
        transition: 200ms;
        padding: 0em;
        span {
            font-size: 1px;
        }
    }

    .inputitem .inputbox:focus ~ .getbuttonbox {
        transform: scaleX(0);
        transition: 200ms;
        padding: 0em;
        span {
            font-size: 1px;
        }        
    }

    .resultsdiv {
        display: none;
        .resultsInner {
            width: 100%;
            background-color: azure;
            transition: 200ms;
        }
    }

    .inputwrapper:focus-within .resultsdiv {
        display: inline-block;
        width: 100%;
        border-radius: 0.2em;
        padding: 0.2em;
        padding-top: 0.5em;
        font-size: 1em;
        color: black;
        .resultsInner {
            width: 100%;
            background-color: azure;
        }

        .resultsInner:hover {
            background-color: cadetblue;
            transition: 200ms;
        }

        .resultsInner:active {
            background-color:cornflowerblue;
        }
    }

    .placesholder {
        padding: 1em;
        height: 100vh;
        width: 100%;
        display: flex;
        justify-content: right;
        align-items: flex-end;
    }

    .placesholderinner {
        background-color: rgba(95, 158, 160, 0.5);
        height: 95%;
        overflow-y: auto;
        pointer-events: auto;
        width: 70%;
        border-radius: 5px;
        box-shadow: 10px 5px 5px rgba(47, 48, 48, 0.5);
        display: flex;
        flex-direction: column;
        transition: 200ms;
        opacity: 0.2;
    }

    .placesholderinner:hover {
        opacity: 1;
        transition: 200ms;
    }

    .oneplace {
        padding: 0.5em;
        width: 100%;
    }

    .oneplaceinner {
        background-color: azure;
        border-radius: 5px;
        transition: 200ms;
        width: 100%;
        cursor: pointer;
        display: inline-block;
        text-decoration: none;
        text-align: center;
    }

    .oneplaceinner:hover {
        background-color: cornflowerblue;
        transition: 200ms;
    }

    .image {
        border-radius: 5px;
        width: 100%;
    }

    .placetitle {
        padding: 0.5em;
        font-size: 2em;
        font-weight: 400;
    }

    .placeaddress {
        padding: 0.2em;
        font-size: 1.2em;
        font-weight: 200;
    }
</style>

<div id="map" bind:this={mapelement}></div>

<div class="overlay">
    <div class="inputholder">
        <div class="inputdiv">
            <div class="inputcar">🚗</div>
            <div class="inputwrapper">
                <!-- <h2 class="labels font">Enter your location:</h2>   -->
                <div class="inputitem">
                    <input class="inputbox" type="text" placeholder="Enter your location" id="yourloc" oninput={onyourplacechange}>
                    <div class='getbuttonbox'>
                        <button class='getbuttonfront' id="getyourlocationfront"><span>📍</span></button>
                        <button class='getbuttonback' id="getyourlocation" onclick={onrequestpress}><span>Request Location</span></button>
                    </div>
                </div>
                {#each yoursuggestions as x}
                <div class="resultsdiv">
                    <button class="resultsInner" onclick={() => onpressyou(x.id)}>
                        <span>{x.text}</span>
                    </button>
                </div>
                {/each}
            </div>
            <div class="inputwrapper">
                <!-- <h2 class="labels font">Enter their location:</h2>-->
                <div class="inputitem">
                    <input class="inputboxfull" type="text" id="themloc" placeholder="Enter their location" oninput={ontheirplacechange}>
                </div>
                {#each theirsuggestions as x}
                <div class="resultsdiv">
                    <button class="resultsInner" onclick={() => onpressthem(x.id)}>
                        <span>{x.text}</span>
                    </button>
                </div>
                {/each}
            </div>
            <div class="inputwrapper">
                <div class="inputitem">
                    <button class="submitbutton" onclick={getDirections}>{submitbuttontext}</button>
                </div>
            </div>
            {#if directionresult}
            <div class="inputwrapper">
                <div class="inputitem">
                    <button class="submitbutton" onclick={getMiddle}>Get Middle📍</button>
                </div>
            </div>
            {/if}
            {#if middlemapmarker}
            <div class="inputwrapper">
                <!-- <h2 class="labels font">Enter their location:</h2>-->
                <div class="inputitem">
                    <input class="inputboxfull" type="text" id="midloc" placeholder="Points of interest" oninput={onmidchange}>
                </div>
            </div>
            {/if}
        </div>
    </div>
    <div class="logobox font">
        <div class='logosurround'>MidpointFinder🔍</div>
        <div>©️ bhargav inc</div>
    </div>
    {#if middlemapmarker}
    <div class="placesholder">
        <div class="placesholderinner">
            {#each midpointsuggestions as x}
                <div class="oneplace">
                    <a href={"https://www.google.com/maps/place/?q=place_id:" + x.id} class="oneplaceinner" target="_blank">
                        <div class="placetitle">{x.name}</div>
                        <div class="placeaddress">{x.address}</div>
                        {#if x.photos}
                        <img src={x.photos} alt="" class="image">
                        {/if}
                    </a>
                </div>
            {/each}
        </div>
    </div>
    {/if}
</div>


