<template>
  <div v-if="errorMessage">
    <span style="color: red">{{ errorMessage }}</span
    ><br />
  </div>
  <div v-if="!isAvailability">
    <span>web bluetoothを使用できない環境です</span><br />
    <span>デバイスまたはブラウザを変更してください</span>
  </div>

  <div>
    <span v-if="devices.length > 0" style="text-decoration: underline"
      >接続済みデバイス名</span
    >
    <template v-for="(device, i) in devices" :key="i">
      <p>{{ device.name }}</p>
    </template>
  </div>

  <div>
    <button v-on:click="startDeviceScanner">デバイスを探す</button>
    <!-- <button v-on:click="displayConnectedDevices">確認</button> -->
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";
// import "./style.css";

const isAvailability = ref(true);
const errorMessage = ref();

const devices = ref<BluetoothDevice[]>([]);

const startDeviceScanner = async () => {
  errorMessage.value = undefined;
  isAvailability.value = await navigator.bluetooth.getAvailability();
  if (!isAvailability.value) {
    return;
  }

  /**
    参考資料
    https://webbluetoothcg.github.io/web-bluetooth/#dom-bluetoothadvertisingevent-rssi
  
  */

  // https://github.com/circuitpython/web-editor/issues/40
  // 接続したデバイスを取得
  // 実装するならChromeの設定を変更する必要あり
  // https://developer.mozilla.org/ja/docs/Web/API/Bluetooth/getDevices#%E3%83%96%E3%83%A9%E3%82%A6%E3%82%B6%E3%83%BC%E3%81%AE%E4%BA%92%E6%8F%9B%E6%80%A7
  // const devices = await navigator.bluetooth.getDevices();
  // console.log(devices);
  // From version 85: this feature is behind the #enable-experimental-web-platform-features preference (needs to be set to enabled). To change preferences in Chrome, visit chrome://flags.
  // navigator.bluetooth.onadvertisementreceived = (
  //   e: BluetoothAdvertisingEvent
  // ) => {
  //   console.log(e.rssi);
  // };

  // Bluetoothデバイスをリクエストする
  // const device = await navigator.bluetooth.requestDevice({
  //   acceptAllDevices: true,
  // });
  // device.addEventListener(
  //   "advertisementreceived",
  //   (e: BluetoothAdvertisingEvent) => {
  //     console.log("advertisementreceived: ", e);
  //   }
  // );
  // BluetoothデバイスのサービスとキャラクタリスティックのUUIDを取得する
  navigator.bluetooth
    .requestDevice({ acceptAllDevices: true })
    .then((device) => {
      // デバイスが見つかった場合の処理
      return device.gatt?.connect(); // デバイスに接続する
    })
    .then((server) => {
      // GATTサーバーに接続した場合の処理
      return server?.getPrimaryServices(); // デバイスのプライマリーサービスを取得する
    })
    .then((services) => {
      // プライマリーサービスが取得された場合の処理
      services?.forEach((service) => {
        console.log("サービスのUUID:", service.uuid); // サービスのUUIDをログに表示する
        service.getCharacteristics().then((characteristics) => {
          // サービスのキャラクタリスティックを取得する
          characteristics.forEach((characteristic) => {
            console.log("キャラクタリスティックのUUID:", characteristic.uuid); // キャラクタリスティックのUUIDをログに表示する
          });
        });
      });
    })
    .catch((error) => {
      // エラーハンドリング
      console.error("エラーが発生しました:", error);
    });

  // devices.value.push(device);
  // name.value = device.name ?? ''
  // const service = await device.gatt?.connect()
  // const preServices = await service?.getPrimaryService('00000d00-0000-1000-8000-00805f9b34fb') //0000181A-0000-1000-8000-00805F9B34FB
  // console.log('service:', service, 'preService: ', preServices)

  // navigator.bluetooth
  //   .requestDevice({
  //     acceptAllDevices: true,
  //   })
  //   .then((device) => {
  //     devices.value.push(device);
  //   })
  //   .catch((error: Error) => {
  //     errorMessage.value = error.message;
  //   })
  //   .finally(() => {});
};
</script>

<style>
body {
  margin: 0;
  display: flex;
  place-items: center;
  min-width: 320px;
  min-height: 100vh;
}

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}
</style>
