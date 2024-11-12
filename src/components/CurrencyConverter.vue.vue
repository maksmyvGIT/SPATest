<template>
  <div>
    <h1>Курсы валют</h1>
    <select v-model="selectedCurrency" @change="getCurrencyRate">
      <option v-for="currency in currencyList" :key="currency.CharCode" :value="currency.CharCode">
        {{ currency.Name }} ({{ currency.CharCode }})
      </option>
    </select>
    <p>Текущий курс: {{ currentRate }}</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      currencyList: [],
      selectedCurrency: '',
      currentRate: null,
    };
  },
  methods: {
    async loadCurrencies() {
      try {
        // Заменяем URL на локальный путь
        const xmlPath = new URL('./XML_daily.xml', import.meta.url).href;
        
        const response = await axios.get(xmlPath, {
          headers: {
            'Content-Type': 'application/xml; charset=utf-8',
          },
        });

        // Парсим XML-ответ
        const parser = new DOMParser();
        const xmlDoc = parser.parseFromString(response.data, 'text/xml');

        // Извлекаем элементы Valute
        const valuteNodes = xmlDoc.getElementsByTagName('Valute');

        // Создание массива объектов валют
        this.currencyList = Array.from(valuteNodes).map((node) => {
          return {
            CharCode: node.getElementsByTagName('CharCode')[0]?.textContent || '',
            Name: node.getElementsByTagName('Name')[0]?.textContent || '',
            Nominal: parseFloat(node.getElementsByTagName('Nominal')[0]?.textContent || 1),
            Value: parseFloat(node.getElementsByTagName('Value')[0]?.textContent.replace(',', '.') || 0),
          };
        });

        // Устанавливаем первую валюту по умолчанию и обновляем курс
        if (this.currencyList.length > 0) {
          this.selectedCurrency = this.currencyList[0].CharCode;
          this.getCurrencyRate();
        }

        console.log("Список валют:", this.currencyList); // Логируем массив валют для проверки

      } catch (error) {
        console.error('Ошибка при загрузке данных о валюте:', error);
      }
    },
    getCurrencyRate() {
      const selectedCurrency = this.currencyList.find(currency => currency.CharCode === this.selectedCurrency);
      if (selectedCurrency) {
        this.currentRate = (selectedCurrency.Value / selectedCurrency.Nominal).toFixed(4);
      } else {
        this.currentRate = null;
      }
    },
  },
  mounted() {
    this.loadCurrencies();
  },
};
</script>
  <style scoped>
  .currency-converter {
    max-width: 300px;
    margin: auto;
    text-align: center;
  }
  </style>
  