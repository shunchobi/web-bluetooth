<script setup lang="ts">
const startDeviceScanner = async () => {
  const device = await navigator.bluetooth.requestDevice({
    filters: [{ services: ["72c90001-57a9-4d40-b746-534e22ec9f9e"] }]
  });
  console.log("connected" + device.name)
  const gattServer = await device.gatt?.connect()
  console.log("gatt server")
  console.log(gattServer)
  const services = await gattServer?.getPrimaryServices()
  console.log("services")
  console.log(services)
  if (services == undefined || services?.length == 0) {
    return
  }

  const service = services[0]
  const tmpcha = await service.getCharacteristics('72c90003-57a9-4d40-b746-534e22ec9f9e')
  console.log("tmpchar")
  console.log(tmpcha)
}
  // tmpcha[0].addEventListener('characteristicvaluechanged')
</script>

<template>
 <button v-on:click="startDeviceScanner">Button</button>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
