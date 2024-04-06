<template>
  <div v-if="errorMessage">
    <p style="color: red">{{ errorMessage }}</p>
  </div>
  <div v-if="!isAvailability" style="line-height: 10px">
    <p>web bluetoothを使用できない環境です</p>
    <p>デバイスまたはブラウザを変更してください</p>
  </div>

  <!-- <div>
    <span v-if="devices.length > 0" style="text-decoration: underline"
      >接続済みデバイス名</span
    >
    <template v-for="(device, i) in devices" :key="i">
      <p>{{ device.name }}</p>
    </template>
  </div> -->

  <div v-if="Object.keys(advDevices).length > 0" style="padding: 10px">
    <p>※受信信号の強さが大きいほどデバイスからの距離が近いです</p>
    <p>※受信信号の強さの降順で表示中</p>
    <table>
      <thead>
        <th>デバイス名（デバイス名が不明の場合はデバイスID）</th>
        <th>受信信号の強さ</th>
      </thead>
      <tbody>
        <template v-for="device in sortAdvDevices" :key="device.deviceName">
          <tr>
            <td>{{ device.deviceName }}</td>
            <td>{{ device.rssi }}</td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>

  <div>
    <button v-on:click="test">デバイスを探す</button>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref } from "vue";
import _ from "lodash";
// import "./style.css";

const isAvailability = ref(true);
const errorMessage = ref();
const log = ref("");

const advDevices = reactive<{ [name: string]: string }>({});
const sortAdvDevices = computed(() => {
  const sortArr = [];
  for (const deviceName in advDevices) {
    const rssi = advDevices[deviceName];
    sortArr.push({
      deviceName,
      rssi,
    });
  }

  return _.sortBy(sortArr, "rssi");
});

const devices = ref<BluetoothDevice[]>([]);

onMounted(async () => {
  await checkAvailability();
});

const checkAvailability = async () => {
  errorMessage.value = undefined;
  isAvailability.value = await navigator.bluetooth.getAvailability();
};

const test = () => {
  navigator.bluetooth
    .requestLEScan({
      // filters: [{ namePrefix: "ThermoBeacon" }],
      acceptAllAdvertisements: true,
      keepRepeatedDevices: true,
    })
    .then((scan) => {
      navigator.bluetooth.addEventListener(
        "advertisementreceived",
        (e: BluetoothAdvertisingEvent) => {
          // log.value = e.rssi?.toString() ?? "rssi is undefined";
          const deviceName = e.device.name ?? e.device.id;
          advDevices[deviceName] = e.rssi?.toString() ?? "no rssi";
          console.log("advertisementreceived: ", e);
        }
      );
    })
    .catch((e) => {
      errorMessage.value = e;
      console.log("error catched", e);
    });
};

const scan = async () => {
  navigator.bluetooth
    .requestLEScan({
      acceptAllAdvertisements: true,
      keepRepeatedDevices: true,
    })
    .then((scanner) => {
      console.log(scanner.active);

      navigator.bluetooth.addEventListener("advertisementreceived", (event) => {
        /* Display device data */
        console.log(event.device.name);
        console.log(event.rssi);
        console.log(event.serviceData);
      });
    })
    .catch((error) => {
      console.log(error);
    });
};

const startDeviceScanner = async () => {
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
  // From version 85: this feature is behind the

  // #enable-experimental-web-platform-features
  // Use the new permissions backend for Web Bluetooth

  // preference (needs to be set to enabled). To change preferences in Chrome, visit chrome://flags.
  // navigator.bluetooth.onadvertisementreceived = (
  //   e: BluetoothAdvertisingEvent
  // ) => {
  //   console.log(e.rssi);
  // };

  // Bluetoothデバイスをリクエストする
  const device = await navigator.bluetooth.requestDevice({
    acceptAllDevices: true,
    /**
      温度
      0000181a-0000-1000-8000-00805f9b34fb

      湿度
      0000181b-0000-1000-8000-00805F9B34FB

      device_information
      0000180a-0000-1000-8000-00805f9b34fb
    */
    optionalServices: ["0000180a-0000-1000-8000-00805f9b34fb"],
  });
  device.addEventListener(
    "advertisementreceived",
    (e: BluetoothAdvertisingEvent) => {
      console.log("advertisementreceived", e.rssi);
    }
  );
  const server = await device.gatt?.connect();
  const services = await server?.getPrimaryServices();

  services?.forEach((service) => {
    // 温度
    // キャラクタリスティックのUUID: 00002a23-0000-1000-8000-00805f9b34fb value:  185
    // キャラクタリスティックのUUID: 00002a24-0000-1000-8000-00805f9b34fb value:  77
    // キャラクタリスティックのUUID: 00002a26-0000-1000-8000-00805f9b34fb value:  70
    // キャラクタリスティックのUUID: 00002a27-0000-1000-8000-00805f9b34fb value:  72
    // キャラクタリスティックのUUID: 00002a28-0000-1000-8000-00805f9b34fb value:  83
    // キャラクタリスティックのUUID: 00002a29-0000-1000-8000-00805f9b34fb
    // キャラクタリスティックのUUID: 00002a2a-0000-1000-8000-00805f9b34fb
    // キャラクタリスティックのUUID: 00002a50-0000-1000-8000-00805f9b34fb

    // 湿度
    // キャラクタリスティックのUUID: 00002a23-0000-1000-8000-00805f9b34fb value:  185
    // キャラクタリスティックのUUID: 00002a24-0000-1000-8000-00805f9b34fb value:  77
    // キャラクタリスティックのUUID: 00002a26-0000-1000-8000-00805f9b34fb
    // キャラクタリスティックのUUID: 00002a27-0000-1000-8000-00805f9b34fb
    // キャラクタリスティックのUUID: 00002a28-0000-1000-8000-00805f9b34fb
    // キャラクタリスティックのUUID: 00002a29-0000-1000-8000-00805f9b34fb
    // キャラクタリスティックのUUID: 00002a2a-0000-1000-8000-00805f9b34fb
    // キャラクタリスティックのUUID: 00002a50-0000-1000-8000-00805f9b34fb
    service
      ?.getCharacteristic("00002a24-0000-1000-8000-00805f9b34fb")
      .then((chara) => {
        // chara!.oncharacteristicvaluechanged = (e: Event) => {
        //   console.log("oncharacteristicvaluechanged", e);
        // };
        return chara?.readValue();
      })
      .then((value) => {
        console.log("service uuid", service.uuid);
        console.log("value: ", value?.getUint8(0));
      });
  });

  navigator.bluetooth.addEventListener("advertisementreceived", (event) => {
    /* Display device data */
    console.log(event.device.name);
    console.log(event.rssi);
    console.log(event.serviceData);
  });

  devices.value.push(device);
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
