<template>
  <button v-on:click="click">Button</button>
</template>

<script setup lang="ts">

 const click = async() => {

const device = await navigator.bluetooth.requestDevice({
  filters: [{ services: ["72c90001-57a9-4d40-b746-534e22ec9f9e"] }],
});
console.log("connected" + device.name);
const gattServer = await device.gatt?.connect();
console.log("gatt server");
console.log(gattServer);
const services = await gattServer?.getPrimaryServices();
console.log("services");
console.log(services);
if (services == undefined || services?.length == 0) {
  return;
}

const service = services[0];
service.onservicechanged = (event) => {
  console.log("service service changed");
  console.log(event);
};
const chars = await service.getCharacteristics();
console.log(chars);

const chaIndicate = await service.getCharacteristic(
  "72c90005-57a9-4d40-b746-534e22ec9f9e"
);
console.log("chaIndicate");
console.log(chaIndicate);
await chaIndicate.startNotifications();
console.log("start notification: ok");
chaIndicate.addEventListener("characteristicvaluechanged", (event) => {
  console.log("notification event");
  console.log(event);
});
const indicateDescriptors = await chaIndicate.getDescriptors();
console.log("indicateDescriptors");
console.log(indicateDescriptors);

const chaWrite = await service.getCharacteristic(
  "72c90004-57a9-4d40-b746-534e22ec9f9e"
);
console.log("chaWrite");
console.log(chaWrite);
const feature = new Uint8Array([0x00, 0x02, 0x01, 0x03]);
await chaWrite.writeValue(feature);

const chaNotify = await service.getCharacteristic(
  "72c90003-57a9-4d40-b746-534e22ec9f9e"
);
console.log("chaNotify");
console.log(chaNotify);

await chaNotify.startNotifications();
console.log("start notification: ok");
chaNotify.addEventListener("characteristicvaluechanged", (event) => {
  console.log("notify event");
  console.log(event);
});


// await chaWrite.writeValue(new Uint8Array([0, 2, 1, 3]));
// await chaWrite.writeValue(command2buf([0, 2, 1, 3]));
console.log("ok");





// const click = () => {
  //     navigator.bluetooth.requestDevice({filters: [{ services: ['72c90001-57a9-4d40-b746-534e22ec9f9e'] }] })
  // .then(device => device.gatt!.connect())
  // .then(server => {
  //   return server.getPrimaryService('72c90001-57a9-4d40-b746-534e22ec9f9e')
  // })
  // .then(service => {
  //   return service.getCharacteristic('72c90004-57a9-4d40-b746-534e22ec9f9e')
  // })
  // .then(characteristic => {
  //   0x00020103
  //   characteristic.writeValueWithoutResponse(['0x00', '0x02', '0x01', '0x03']);
  //   return characteristic.startNotifications()})
  // .then(characteristic => {
  //   characteristic.addEventListener('characteristicvaluechanged', handleCharacteristicValueChanged);
  //   console.log('Notifications have been started. : ' + Object.keys(characteristic));
  //   // return characteristic.readValue();
  // })
  // .catch(error => { console.error(error); });
  // function handleCharacteristicValueChanged(event: any) {
  //   const value = event.target.value;
  //   console.log('Received ' + value);
  //   console.log(event)
  //   const batteryLevel = event.target.value.getUint8(0);
  //   console.log('Battery percentage is ' + batteryLevel);
  //   // TODO: Heart Rate Measurement値を解析する。
  //   // https://github.com/WebBluetoothCG/demos/blob/gh-pages/heart-rate-sensor/heartRateSensor.jsを参照
  // }
  // };
  // const str2ab = (str: string) => {
  //   var buf = new ArrayBuffer(str.length * 2); // 2 bytes for each char
  //   var bufView = new Uint16Array(buf);
  //   for (var i = 0, strLen = str.length; i < strLen; i++) {
  //     bufView[i] = str.charCodeAt(i);
  //   }
  //   return buf;
};
const safaf = 'sfada'
</script>

<style scoped></style>
