<template>
  <button v-on:click="clicked">Button</button>
</template>

<script setup lang="ts">
  import BluetoothDevice from "web-bluetooth"
    
const clicked = () => {
  // const BluetoothDevice = require("web-bluetooth");

  let exampleDevice = new BluetoothDevice({
    uuid: "72c90001-57a9-4d40-b746-534e22ec9f9e",  // 72c90005-57a9-4d40-b746-534e22ec9f9e（Indicate）
  });

  exampleDevice.connect();

  if (exampleDevice.connected()) {
    console.log("status: " + exampleDevice.connected());
  }

  exampleDevice
    .getValue("battery_level")
    .then((value: { battery_level: any }) => {
      console.log(value.battery_level);
    });

  exampleDevice.startNotifications("72c90003-57a9-4d40-b746-534e22ec9f9e", (eventObj: { heart_rate_measurement: any; }) => {
    var newHR = eventObj.heart_rate_measurement;
    console.log(newHR);
  });
};
</script>

<style scoped></style>
