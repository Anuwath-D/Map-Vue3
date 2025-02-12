<template>
  <div class="mapdetail">
    <div class="box">
      <img src="../assets/iconmap.svg" alt="Logo" width="24" />
      <div class="font">แผนที่เเสดงเส้นทางการเดินรถ  </div>
      <div>(กดที่เเผนที่เพื่อปักหมุดจุดที่ต้องการคำนวณ)</div>
    </div>

    <!-- map -->
    <div id="map"></div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import L from "leaflet";
import axios from "axios";
import people from "@/assets/people.svg";
import busGreen from "@/assets/bus-green.svg";

// สร้างตัวแปรเพื่อเก็บตำแหน่งของผู้ใช้
const currentLat = ref(null);
const currentLng = ref(null);
const minutes = ref(0);
const seconds = ref(0);
const distanceInKm = ref(0);
const speedInKmHr = ref(0);

// ฟังก์ชันดึงตำแหน่งผู้ใช้
function getUserLocation() {
  return new Promise((resolve, reject) => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          currentLat.value = position.coords.latitude;
          currentLng.value = position.coords.longitude;
          resolve(); // เสร็จสิ้นการดึงตำแหน่ง
        },
        (error) => {
          reject("ไม่สามารถดึงตำแหน่งได้: " + error.message);
        }
      );
    } else {
      reject("เบราว์เซอร์นี้ไม่รองรับ Geolocation API");
    }
  });
}

async function getRouteData(currentlat, currentLng, lat, lng) {
  const url = `https://router.project-osrm.org/route/v1/driving/${currentLng},${currentlat};${lng},${lat}?overview=full&geometries=geojson`;

  // console.log("URL:", url);

  try {
    const response = await axios.get(url);
    console.log("Response:", response.data);

    // ตรวจสอบว่ามีข้อมูลเส้นทางหรือไม่
    if (response.data.routes && response.data.routes[0]) {
      console.log("response", response.data.routes[0].distance);

      speedInKmHr.value = 40; // ความเร็วในหน่วยกิโลเมตรต่อชั่วโมง
      distanceInKm.value = response.data.routes[0].distance / 1000; // แปลงระยะทางเป็นกิโลเมตร
      const timeInHours = distanceInKm.value / speedInKmHr.value; // คำนวณเวลาในชั่วโมง
      const timeInMinutes = timeInHours * 60; // แปลงเวลาเป็นนาที

      minutes.value = Math.floor(timeInMinutes)
      seconds.value = Math.round((timeInMinutes - minutes.value) * 60)

      console.log(
        `เวลาเดินทางประมาณ: ${minutes.value} นาที ${seconds.value} วินาที`
      );
    } else {
      console.error("ไม่สามารถคำนวณเส้นทางได้");
    }
  } catch (error) {
    console.error("Error fetching route data:", error);
  }
}

onMounted(async () => {
  // เรียกใช้ฟังก์ชันดึงตำแหน่ง
  try {
    await getUserLocation();
    console.log("ตำแหน่งผู้ใช้:", currentLat.value, currentLng.value);

    // สร้างแผนที่และตั้งค่าตำแหน่งเริ่มต้น
    const map = L.map("map").setView(
      [currentLat.value || 13.736717, currentLng.value || 100.523186],
      12
    );

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors",
    }).addTo(map);

    let currentMarker = null; // เก็บ marker ที่ถูกปักไว้

    map.on("click", async function (e) {
      const { lat, lng } = e.latlng;

      // ถ้ามี marker อยู่แล้ว ให้ลบออกก่อน
      if (currentMarker) {
        map.removeLayer(currentMarker);
      }

      L.Marker.prototype.options.icon = L.icon({
        iconUrl: busGreen,
        shadowUrl:
          "https://unpkg.com/leaflet@1.9.4/dist/images/marker-shadow.png",
        iconSize: [30, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      });

      await getRouteData(currentLat.value, currentLng.value, lat, lng);

      // ปัก marker ใหม่
      currentMarker = L.marker([lat, lng])
        .addTo(map)
        .bindPopup(
          `Latitude : ${lat.toFixed(5)}, Longitude : ${lng.toFixed(5)}<br>
          ระยะทาง : ${distanceInKm.value.toFixed(2)} km<br>
          ความเร็วเฉลี่ย : ${speedInKmHr.value} km/hr<br>
          ระยะเวลาการเดินทาง : ${minutes.value} นาที`
        )
        .openPopup();
    });

    // ถ้ามีตำแหน่งผู้ใช้ ให้ปักหมุดที่ตำแหน่งนั้น
    if (currentLat.value && currentLng.value) {
      L.Marker.prototype.options.icon = L.icon({
        iconUrl: people,
        shadowUrl:
          "https://unpkg.com/leaflet@1.9.4/dist/images/marker-shadow.png",
        iconSize: [30, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      });

      L.marker([currentLat.value, currentLng.value])
        .addTo(map)
        .bindPopup("ตำแหน่งของคุณ")
        .openPopup();
    }
  } catch (error) {
    console.error(error); // หากเกิดข้อผิดพลาดในการดึงตำแหน่ง
  }
});
</script>

<style scoped>
.mapdetail {
  background: #ffffff;
  padding: 16px;
  border-radius: 16px;
  height: 85vh;
  box-shadow: 0px 4px 4px 0px rgba(0, 0, 0, 0.25);
}

.font {
  font-size: 16px;
  font-weight: 600;
}

.box {
  display: flex;
  gap: 12px;
  align-items: center;
}

#map {
  width: 100%;
  height: 600px;
  margin-top: 18px;
  border-radius: 10px;
}
</style>