<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="primary">
          <ion-button @click="refreshData()">
            <ion-icon slot="icon-only" :icon="refresh"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-title class="ion-text-center">Cryptocurrency</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large" class="ion-text-center">Cryptocurrency</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-grid>
        <ion-row class="header-row">
          <ion-col size="2"><b>Rank</b></ion-col>
          <ion-col size="6"><b>Name (Symbol)</b></ion-col>
          <ion-col size="4" class="ion-text-right"><b>Price (USD)</b></ion-col>
        </ion-row>

        <ion-row v-for="crypto in cryptos" :key="crypto.id" class="data-row">
          <ion-col size="2">{{ crypto.rank }}</ion-col>
          <ion-col size="6">{{ crypto.name }} ({{ crypto.symbol }})</ion-col>
          <ion-col size="4" class="ion-text-right">${{ crypto.price_usd }}</ion-col>
        </ion-row>
      </ion-grid>

      <ion-infinite-scroll @ionInfinite="loadMore($event)">
        <ion-infinite-scroll-content loading-spinner="bubbles" loading-text="Loading more data..."></ion-infinite-scroll-content>
      </ion-infinite-scroll>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent, 
  IonGrid,
  IonRow,
  IonCol,
  IonButtons,
  IonButton,
  IonIcon,
  IonInfiniteScroll,
  IonInfiniteScrollContent,
  InfiniteScrollCustomEvent
} from '@ionic/vue';
import { ref, onMounted } from 'vue';
import { refresh } from 'ionicons/icons';

interface Crypto {
  id: string;
  symbol: string;
  name: string;
  rank: number;
  price_usd: string;
}

const cryptos = ref<Crypto[]>([]);
const start = ref(0);

const fetchCryptos = async (startVal: number = 0) => {
  try {
    const response = await fetch(`https://api.coinlore.net/api/tickers/?start=${startVal}&limit=50`);
    const json = await response.json();
    if (startVal === 0) {
      cryptos.value = json.data;
    } else {
      cryptos.value = [...cryptos.value, ...json.data];
    }
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};

const refreshData = async () => {
  start.value = 0;
  await fetchCryptos(0);
};

const loadMore = async (ev: InfiniteScrollCustomEvent) => {
  start.value += 50;
  await fetchCryptos(start.value);
  ev.target.complete();
};

onMounted(() => {
  fetchCryptos();
});
</script>

<style scoped>
.header-row {
  border-bottom: 2px solid var(--ion-color-medium);
  background-color: var(--ion-color-light);
  position: sticky;
  top: 0;
  z-index: 10;
}

.data-row {
  border-bottom: 1px solid var(--ion-color-light-shade);
  align-items: center;
}

.ion-text-right {
  text-align: right;
}

.ion-text-center {
  text-align: center;
}
</style>
