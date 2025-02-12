<template>
  <div class="mapdetail">
    <div class="box">
      <img src="../assets/iconmap.svg" alt="Logo" width="24" />
      <div class="font">แผนที่เเสดงตำเเหน่งรถ</div>
    </div>

    <!-- map -->
    <div id="map"></div>
  </div>
</template>

<script>
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import busRed from "@/assets/bus-red.svg";
import busYellow from "@/assets/bus-yellow.svg";
import busGreen from "@/assets/bus-green.svg";
import busBlue from "@/assets/bus-blue.svg";
import busGrey from "@/assets/bus-grey.svg";

export default {
  name: "Mapmarker",
  data() {
    return {
      map: null,
      current_Lat: null,
      current_Long: null,
      alldriver: [
        {
          id: 1,
          latitude: 7.026852,
          longitude: 100.467532,
          phone_num: 10,
          reg_num: 103,
          dis: 10,
          avg: 58,
          full_student: 30,
          count_studennt: 30,
          status: "online",
        },
        {
          id: 2,
          latitude: 7.002441,
          longitude: 100.529912,
          phone_num: 20,
          reg_num: 202,
          dis: 20,
          avg: 82,
          full_student: 30,
          count_studennt: 15,
          status: "online",
        },
        {
          id: 3,
          latitude: 6.974898,
          longitude: 100.479213,
          phone_num: 30,
          reg_num: 301,
          dis: 60,
          avg: 65,
          full_student: 30,
          count_studennt: 10,
          status: "online",
        },
        {
          id: 4,
          latitude: 7.004536,
          longitude: 100.415211,
          phone_num: 50,
          reg_num: 403,
          dis: 50,
          avg: 0,
          full_student: 30,
          count_studennt: 0,
          status: "offline",
        },
        {
          id: 5,
          latitude: 7.005913,
          longitude: 100.497579,
          phone_num: 50,
          reg_num: 502,
          dis: 50,
          avg: 0,
          full_student: 30,
          count_studennt: 0,
          status: "spare",
        },
        {
          id: 6,
          latitude: 6.979642,
          longitude: 100.4648,
          phone_num: 0,
          reg_num: 601,
          dis: 30,
          avg: 88,
          full_student: 30,
          count_studennt: 22,
          status: "online",
        },
        {
          id: 7,
          latitude: 6.998504,
          longitude: 100.447515,
          phone_num: 50,
          reg_num: 50,
          dis: 30,
          avg: 55,
          full_student: 30,
          count_studennt: 8,
          status: "online",
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
      if (this.map) {
        this.map.remove(); // Destroys the existing map instance
        this.map = null;
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
            12
          );

          L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            maxZoom: 19,
            tileSize: 512,
            zoomOffset: -1,
            attribution: "© OpenStreetMap",
            detectRetina: true,
          }).addTo(this.map);

          // วนลูปสร้าง marker สำหรับข้อมูลใน this.alldriver
          for (const element of this.alldriver) {
            const iconHtml = `
                <div style="position: relative; text-align: center;">
                  <img src="${
                    element.avg >= 80
                      ? busRed
                      : element.avg >= 60
                      ? busYellow
                      : element.avg > 0
                      ? busGreen
                      : element.avg == 0 && element.status == "spare"
                      ? busBlue
                      : busGrey
                  }" style="display: block; width: 32px;" />
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

            const marker = L.marker([element.latitude, element.longitude], {
              icon: icon,
            })
              .addTo(this.map) // Add marker
              .bindPopup(popupContent);

            marker.on("popupopen", () => {
              setTimeout(() => {
                const button = document.getElementById(`btn-${element.id}`);
                if (button) {
                  button.addEventListener("click", () => {
                    this.router.navigate([
                      `/map-dlt/vehicle-data/${element.id}`,
                    ]);
                  });
                }
              }, 0);
            });
          }
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
    // ฟังก์ชันที่ใช้ดึงตำแหน่งปัจจุบัน
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