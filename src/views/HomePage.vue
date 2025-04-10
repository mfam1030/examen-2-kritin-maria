<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar color="primary">
        <ion-title class="ion-text-center">Lector QR</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <div class="scan-container">
        <ion-button 
          expand="block" 
          @click="scanBarcode"
          class="scan-button"
          shape="round"
        >
          <ion-icon slot="start" :icon="scanOutline"></ion-icon>
          Escanear Código
        </ion-button>
      </div>

      <ion-card v-if="scanHistory.length > 0" class="history-card">
        <ion-card-header>
          <ion-card-title>Historial de Escaneos</ion-card-title>
        </ion-card-header>
        
        <ion-card-content>
          <ion-list lines="none">
            <ion-item 
              v-for="(item, index) in scanHistory" 
              :key="index"
              class="history-item"
            >
              <ion-icon :icon="qrCodeOutline" slot="start" color="medium"></ion-icon>
              <ion-label class="ion-text-wrap">{{ item }}</ion-label>
              <ion-button 
                fill="clear" 
                size="small"
                @click="handleItemAction(item)"
              >
                <ion-icon :icon="openOutline"></ion-icon>
              </ion-button>
            </ion-item>
          </ion-list>
        </ion-card-content>
      </ion-card>

      <div v-else class="empty-state">
        <ion-icon :icon="qrCodeOutline" class="empty-icon"></ion-icon>
        <h3>No hay escaneos recientes</h3>
        <p>Presiona el botón para escanear tu primer código</p>
      </div>

      <ion-alert
        :is-open="showAlert"
        :header="'Resultado del escaneo'"
        :message="alertMessage"
        :buttons="['OK']"
        @didDismiss="showAlert = false"
      ></ion-alert>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { 
  IonButton, IonContent, IonHeader, IonPage, IonTitle, IonToolbar, 
  IonAlert, IonList, IonItem, IonLabel, IonIcon, IonCard, IonCardHeader,
  IonCardTitle, IonCardContent
} from '@ionic/vue';
import { scanOutline, qrCodeOutline, openOutline } from 'ionicons/icons';
import { CapacitorBarcodeScanner } from '@capacitor/barcode-scanner';

const showAlert = ref(false);
const alertMessage = ref('');
const scanHistory = ref<string[]>([]);

async function scanBarcode() {
  try {
    const result = await CapacitorBarcodeScanner.scanBarcode({ hint: 17 });

    if (result.ScanResult) {
      const content = result.ScanResult;
      scanHistory.value.unshift(content);
      handleItemAction(content);
    } else {
      alertMessage.value = 'No se detectó contenido en el código QR.';
      showAlert.value = true;
    }
  } catch (error) {
    alertMessage.value = 'Error al escanear el código QR.';
    showAlert.value = true;
    console.error(error);
  }
}

function handleItemAction(content: string) {
  if (content.includes('@') && content.includes('.')) {
    window.location.href = `mailto:${content}`;
  } else if (content.startsWith('http')) {
    window.open(content, '_blank');
  } else {
    alertMessage.value = content;
    showAlert.value = true;
  }
}
</script>

<style scoped>
.scan-container {
  margin: 2rem 0;
  text-align: center;
}

.scan-button {
  --ion-color-primary: #3880ff;
  --ion-color-primary-contrast: #ffffff;
  height: 50px;
  font-weight: bold;
  margin: 0 auto;
  max-width: 300px;
}

.history-card {
  border-radius: 12px;
  margin-top: 1.5rem;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}

.history-item {
  --padding-start: 0;
  --inner-padding-end: 0;
  margin-bottom: 8px;
  border-radius: 8px;
  transition: background-color 0.2s;
}

.history-item:hover {
  background-color: rgba(56, 128, 255, 0.1);
}

.empty-state {
  text-align: center;
  margin-top: 3rem;
  color: var(--ion-color-medium);
}

.empty-icon {
  font-size: 64px;
  opacity: 0.5;
  margin-bottom: 1rem;
}

.empty-state h3 {
  font-weight: 500;
  margin-bottom: 0.5rem;
}

.empty-state p {
  margin-top: 0;
  font-size: 0.9rem;
}
</style>