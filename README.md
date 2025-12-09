# OFFICE
239
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>行程規劃 Web App</title>
    <!-- 引入 Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 引入 Vue 3 CDN -->
    <script src="unpkg.com"></script>
    <style>
        /* 確保地圖容器有指定高度 */
        #map {
            height: 100%;
            width: 100%;
        }
        /* 使整個應用程式填滿視窗高度 */
        html, body, #app {
            height: 100%;
            margin: 0;
            padding: 0;
        }
    </style>
</head>
<body>
    <div id="app" class="flex h-screen">
        <!-- 側邊欄：行程列表 -->
        <div class="w-1/3 bg-white p-4 shadow-lg overflow-y-auto">
            <h1 class="text-2xl font-bold mb-4">我的行程</h1>
            <div class="mb-4">
                <input v-model="newPlace" @keyup.enter="addPlace" type="text" placeholder="輸入地點名稱" class="p-2 border border-gray-300 rounded w-full mb-2">
                <button @click="addPlace" class="bg-blue-500 text-white p-2 rounded w-full hover:bg-blue-600">新增地點</button>
            </div>
            <ul class="space-y-2">
                <li v-for="(place, index) in itinerary" :key="index" class="flex justify-between items-center p-3 bg-gray-100 rounded shadow-sm">
                    <span>{{ index + 1 }}. {{ place.name }}</span>
                    <button @click="removePlace(index)" class="text-red-500 hover:text-red-700">移除</button>
                </li>
            </ul>
        </div>
        
        <!-- 地圖區域 -->
        <div class="w-2/3" id="map"></div>
    </div>

    <script>
        const { createApp, ref, onMounted, watch } = Vue;

        const GOOGLE_MAPS_API_KEY = 'YOUR_API_KEY'; // **請替換為您的 API 金鑰**
        const initialCoords = { lat: 23.634, lng: 120.973 }; // 台灣中心座標

        const app = createApp({
            setup() {
                const newPlace = ref('');
                const itinerary = ref([]);
                let map = null;
                const markers = ref([]);

                // 載入 Google Maps API
                const loadGoogleMapsScript = () => {
                    if (window.google) {
                        initMap();
                        return;
                    }
                    const script = document.createElement('script');
                    script.src = `maps.googleapis.com{GOOGLE_MAPS_API_KEY}&callback=initMap`;
                    script.async = true;
                    script.defer = true;
                    document.head.appendChild(script);
                };

                // 初始化地圖
                const initMap = () => {
                    map = new google.maps.Map(document.getElementById('map'), {
                        center: initialCoords,
                        zoom: 8,
                    });
                    // 將 initMap 綁定到 window，以便 API callback 呼叫
                    window.initMap = initMap; 
                };

                // 新增地點到行程
                const addPlace = () => {
                    if (newPlace.value.trim() === '') return;
                    
                    // 使用 Geocoding Service 尋找地點座標
                    const geocoder = new google.maps.Geocoder();
                    geocoder.geocode({ 'address': newPlace.value }, (results, status) => {
                        if (status === 'OK' && results[0]) {
                            const location = results[0].geometry.location;
                            itinerary.value.push({
                                name: newPlace.value,
                                lat: location.lat(),
                                lng: location.lng(),
                            });
                            newPlace.value = '';
                        } else {
                            alert('找不到該地點，請輸入更詳細的地址。');
                        }
                    });
                };

                // 移除行程中的地點
                const removePlace = (index) => {
                    itinerary.value.splice(index, 1);
                };

                // 監聽行程變化，更新地圖標記和路線
                watch(itinerary, (newItinerary) => {
                    // 清除現有標記
                    markers.value.forEach(marker => marker.setMap(null));
                    markers.value = [];

                    if (newItinerary.length > 0) {
                        // 加入新標記
                        newItinerary.forEach(place => {
                            const marker = new google.maps.Marker({
                                position: { lat: place.lat, lng: place.lng },
                                map: map,
                                title: place.name,
                            });
                            markers.value.push(marker);
                        });

                        // 繪製路線 (簡單的多點連線)
                        if (newItinerary.length > 1) {
                            const pathCoords = newItinerary.map(p => ({ lat: p.lat, lng: p.lng }));
                            const itineraryPath = new google.maps.Polyline({
                                path: pathCoords,
                                geodesic: true,
                                strokeColor: '#FF0000',
                                strokeOpacity: 1.0,
                                strokeWeight: 2
                            });
                            itineraryPath.setMap(map);
                        }

                        // 自動調整地圖中心以包含所有標記
                        const bounds = new google.maps.LatLngBounds();
                        newItinerary.forEach(place => bounds.extend({ lat: place.lat, lng: place.lng }));
                        map.fitBounds(bounds);
                    }
                }, { deep: true });

                onMounted(() => {
                    loadGoogleMapsScript();
                });

                return {
                    newPlace,
                    itinerary,
                    addPlace,
                    removePlace
                };
            }
        });

        app.mount('#app');
    </script>
</body>
</html>
