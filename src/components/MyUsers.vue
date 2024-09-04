<template>
  <div class="container">
    <!-- BarchartData bileşeni -->
    <BarchartData :latitudes="users.map(user => parseFloat(user.address.geo.lat || 0))" />

    <!-- Yeni eklenen tablo (Pagination ile) -->
    <div class="table-container">
      <table class="table">
        <thead>
          <tr>
            <th scope="col">SKU</th>
            <th scope="col">Product Name</th>
            <th scope="col">Net Fee</th>
            <th scope="col">Percentage</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in paginatedUsers" :key="user.id">
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td :class="getNetFeeClass(calculateNetFee(user))">
              {{ calculateNetFee(user) }}
            </td> 
            <td :class="getPercentageClass(user)">
              {{ calculatePercentage(user) }}%
            </td> 
          </tr>
        </tbody>
      </table>

      <!-- Pagination kontrolü -->
      <nav>
        <ul class="pagination">
          <li 
            class="page-item" 
            :class="{ disabled: currentPage === 1 }"
            @click.prevent="currentPage > 1 && currentPage--"
          >
            <a class="page-link" href="#">←</a>
          </li>
          <li class="page-item active">
            <span class="page-link">{{ currentPage }}</span>
          </li>
          <li 
            class="page-item" 
            :class="{ disabled: currentPage === totalPages }"
            @click.prevent="currentPage < totalPages && currentPage++"
          >
            <a class="page-link" href="#">→</a>
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import BarchartData from './BarchartData.vue'; 

export default {
  components: {
    BarchartData
  },
  data() {
    return {
      users: null,
      currentPage: 1,
      perPage: 3, // Her sayfada gösterilecek veri sayısı
    };
  },
  created() {
    axios.get("https://jsonplaceholder.typicode.com/users").then(res => {
      this.users = res.data;
      console.log(this.users);
    });
  },
  computed: {
    totalLatitude() {
      if (!this.users) return 0;

      const total = this.users.reduce((acc, user) => {
        const lat = parseFloat(user.address?.geo?.lat || 0);
        return acc + lat;
      }, 0);

      return total;
    },
    paginatedUsers() {
      const start = (this.currentPage - 1) * this.perPage;
      const end = start + this.perPage;
      return this.users ? this.users.slice(start, end) : [];
    },
    totalPages() {
      // Toplam sayfa sayısı
      return this.users ? Math.ceil(this.users.length / this.perPage) : 0;
    }
  },
  methods: {
    changePage(page) {
      this.currentPage = page;
    },
    calculateNetFee(user) {

      const lat = parseFloat(user.address?.geo?.lat || 0);
      const lng = parseFloat(user.address?.geo?.lng || 0);
      return (lat - lng).toFixed(2); 
    },
    getNetFeeClass(netFee) {
      return netFee < 0 ? 'text-danger' : 'text-success';
    },
    calculatePercentage(user) {
      // lat ve lng değerlerinin oranını hesapla ve yüzde olarak döndür
      const lat = parseFloat(user.address?.geo?.lat || 0);
      const lng = parseFloat(user.address?.geo?.lng || 0);
      if (lat === 0) return '0.00'; // Sıfır olamaz
      return ((lng / lat) * 100).toFixed(2); // Yüzde olarak hesapla
    },
    getPercentageClass(user) {
      const percentage = this.calculatePercentage(user);
      return percentage < 0 ? 'text-danger' : 'text-success';
    }
  }
};
</script>

<style scoped>
.table-container {
  margin-top: 20px;
}
.pagination {
  display: flex;
  justify-content: flex-end; 
  margin-top: 10px;
}
.page-item {
  cursor: pointer;
}
.page-item.disabled .page-link {
  pointer-events: none;
  color: #ccc;
}
.page-item.active .page-link {
  background-color: #007bff;
  color: white;
}
.text-danger {
  color: red;
}
.text-success {
  color: green;
}
</style>
