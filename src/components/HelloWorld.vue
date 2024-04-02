<template>
  <p style="font-size: large">
    {{
      name != undefined
        ? `デバイス名：${name}`
        : "接続したデバイスの名前が見つかりませんでした"
    }}
  </p>
  <button v-on:click="startDeviceScanner">デバイスを探して接続する</button>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";

const device = ref<BluetoothDevice>();

const name = computed(() => {
  return device.value
});
const startDeviceScanner = async () => {
  // https://github.com/circuitpython/web-editor/issues/40
  // const devices = await navigator.bluetooth.getDevices();
  // console.log(devices);

  device.value = await navigator.bluetooth.requestDevice({
    acceptAllDevices: true,
  });

  // const gattServer = await device.gatt?.connect();

  // console.log("gatt server: ", gattServer);

  // const services = await gattServer?.getPrimaryServices();
  // console.log(services);

  // if (services == undefined || services?.length == 0) {
  //   return;
  // }

  // const service = services[0];
  // const tmpcha = await service.getCharacteristics(
  //   "72c90003-57a9-4d40-b746-534e22ec9f9e"
  // );
  // console.log("tmpchar");
  // console.log(tmpcha);
};
// tmpcha[0].addEventListener('characteristicvaluechanged')
</script>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
