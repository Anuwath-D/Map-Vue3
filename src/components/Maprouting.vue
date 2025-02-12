<template>
  <div class="mapdetail">
    <div class="box">
      <img src="../assets/iconmap.svg" alt="Logo" width="24" />
      <div class="font">แผนที่เเสดงเส้นทางการเดินรถ</div>
    </div>

    <!-- map -->
    <div id="map"></div>
  </div>
</template>

<script>
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import axios from "axios";
import busYellow from "@/assets/bus-yellow.svg";
import busGreen from "@/assets/bus-green.svg";
import busGrey from "@/assets/bus-grey.svg";
import people from "@/assets/people.svg";

export default {
  name: "Maprouting",
  data() {
    return {
      map: null,
      polyline: [],

      current_Lat: null,
      current_Long: null,

      minutes: 0,
      seconds: 0,

      alldriver: [
        {
          id: 1,
          latitude: 7.002441,
          longitude: 100.529912,
          phone_num: 20,
          reg_num: 20,
          dis: 20,
          avg: 20,
          full_student: 30,
          count_studennt: 15,
        },
      ],

      race_data: [
        {
          latitude: 7.001799,
          longitude: 100.500292,
          speed: 20,
          status: "up",
          count: 1,
        },
        {
          latitude: 6.999833,
          longitude: 100.507229,
          speed: 40,
          status: "down",
          count: 3,
        },
        {
          latitude: 6.997537,
          longitude: 100.512393,
          speed: 60,
          status: "up",
          count: 4,
        },
        {
          latitude: 7.000281,
          longitude: 100.520641,
          speed: 70,
          status: "up",
          count: 3,
        },
        {
          latitude: 7.001465,
          longitude: 100.525269,
          speed: 100,
          status: "up",
          count: 2,
        },
        {
          latitude: 7.002566,
          longitude: 100.52948,
          speed: 68,
          status: "down",
          count: 2,
        },
      ],
    };
  },
  mounted() {
    this.initMap();
  },
  beforeUnmount() {
    if (this.map) {
      this.map.remove();
      this.map = null;
    }
  },
  methods: {
    async initMap() {
      // หากแผนที่มีอยู่แล้ว ให้ลบออกก่อน
      if (this.map) {
        this.map.remove(); // Destroys the existing map instance
      }

      if (this.alldriver) {
        try {
          // ใช้ await กับ getUserLocation เพื่อให้แน่ใจว่าตำแหน่งถูกดึงก่อนตั้งแผนที่

          // await this.getUserLocation();

          // console.log(
          //   `ตำแหน่งปัจจุบัน2: ${this.current_Lat}, ${this.current_Long}`
          // );

          // กำหนดตำแหน่งแผนที่เริ่มต้นจากข้อมูลใน this.alldriver
          this.map = L.map("map").setView(
            [7.005994, 100.5024677],
            14
          );

          L.Marker.prototype.options.icon = L.icon({
            iconUrl: people,
            shadowUrl:
              "https://unpkg.com/leaflet@1.9.4/dist/images/marker-shadow.png",
            iconSize: [30, 41],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            shadowSize: [41, 41],
          });

          L.marker([7.005994, 100.5024677]).addTo(this.map);

          L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            maxZoom: 19,
            tileSize: 512,
            zoomOffset: -1,
            attribution: "© OpenStreetMap",
            detectRetina: true,
          }).addTo(this.map);

          // วนลูปสร้าง marker สำหรับข้อมูลใน this.alldriver
          for (const element of this.race_data) {
            // await this.getRouteDistance(element.latitude, element.longitude)

            const iconHtml = `
            <div style="position: relative; text-align: center; font-family: 'Noto Sans Thai';">
              <img src="${
                element.speed === element.speed
                  ? busGreen
                  : element.speed >= 1
                  ? busYellow
                  : busGrey
              }" style="display: block;width : 30px;" />
            </div>
          `;

            const icon = L.divIcon({
              html: iconHtml,
              shadowUrl:
                "https://unpkg.com/leaflet@1.9.4/dist/images/marker-shadow.png",
              iconSize: [25, 41],
              iconAnchor: [12, 41],
              popupAnchor: [1, -34],
              shadowSize: [41, 41],
              className: "custom-leaflet-div-icon",
            });

            const popupContent = this.createPopupContent(element);

            L.marker([element.latitude, element.longitude], { icon: icon })
              .addTo(this.map) // Add marker
              .bindPopup(popupContent);
          }

          this.bus_route();
        } catch (error) {
          console.error("ไม่สามารถดึงตำแหน่งผู้ใช้:", error);
          // กำหนดแผนที่เริ่มต้นที่ตำแหน่งอื่นเมื่อเกิดข้อผิดพลาด
          this.map = L.map("map").setView([7.01, 100.48], 13);
          L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            maxZoom: 19,
            tileSize: 512,
            zoomOffset: -1,
            attribution: "© OpenStreetMap",
            detectRetina: true,
          }).addTo(this.map);
        }
      } else {
        // หาก this.alldriver ไม่มี ให้แสดงแผนที่เริ่มต้น
        this.map = L.map("map").setView([7.01, 100.48], 13);
        L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
          maxZoom: 19,
          tileSize: 512,
          zoomOffset: -1,
          attribution: "© OpenStreetMap",
          detectRetina: true,
        }).addTo(this.map);
      }
    },

    getUserLocation() {
      return new Promise((resolve, reject) => {
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition(
            (position) => {
              this.current_Lat = position.coords.latitude;
              this.current_Long = position.coords.longitude;
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
    },

    async bus_route() {
      let row = {
        latitude: 7.005994, 
        longitude: 100.5024677,
        speed: 0,
        status: "",
        count: 0,
      };
      this.race_data.unshift(row);

      for (let i = 0; i < this.race_data.length - 1; i++) {
        const element = this.race_data[i];
        const nextElement = this.race_data[i + 1];

        await this.getRouteData(element, nextElement);
      }
    },

    async getRouteData(element, nextElement) {
      const url = `https://router.project-osrm.org/route/v1/driving/${element.longitude},${element.latitude};${nextElement.longitude},${nextElement.latitude}?overview=full&geometries=geojson`;

      // console.log("URL:", url);

      try {
        const response = await axios.get(url);
        // console.log("Response:", response.data);

        if (response.data.routes.length > 0) {
          const route = response.data.routes[0].geometry.coordinates.map(
            (coord) => [coord[1], coord[0]]
          );

          L.polyline(route, {
            color:
              nextElement.speed >= 80
                ? "red"
                : nextElement.speed >= 60
                ? "orange"
                : "green",
          }).addTo(this.map);
        }
      } catch (error) {
        console.error("Error fetching route data:", error);
      }
    },

    createPopupContent(element) {
      return `
    <div>
      <h3>รถหมายเลข: ${element.reg_num}</h3>
      <p>สถานะ: ${element.status}</p>
      <p>จำนวนนักเรียน: ${element.count_studennt}/${element.full_student}</p>
    </div>
  `;
    },
  },
};
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