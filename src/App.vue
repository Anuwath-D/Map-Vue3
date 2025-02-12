<template>
  <v-app>
    <v-container>
      <v-app-bar :elevation="2">
        <!-- <template v-slot:prepend>
          <v-app-bar-nav-icon></v-app-bar-nav-icon>
        </template> -->

        <v-app-bar-title style="font-size:26px;">School Bus</v-app-bar-title>

        <v-tabs>
          <v-tab to="/Maptime">คำนวณเวลา</v-tab>
          <v-tab to="/">ตำเเหน่งรถ</v-tab>
          <v-tab to="/Maprouting">เส้นทางการเดินรถ</v-tab>
        </v-tabs>
      </v-app-bar>

      <router-view></router-view>
    </v-container>
  </v-app>
</template>

<script setup>
//
</script>

<style scoped>
.v-container {
  max-width: none;
  padding: 80px 36px;
  background: #f7f8fc;
  height: 100vh;
}
</style>