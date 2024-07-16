<template>
  <div class="content">
    <h1 style="background: rgba(255, 255, 255, 0.8); padding-bottom: 8px;">Welcome to My Finance App</h1>
    <div class="button-container">
      <button @click="openUangMasukModal" class="filter-button">
        <i class="fas fa-plus-circle"></i> Uang Masuk
      </button>
      <button @click="openModal" class="tambah-data-button">
        <i class="fas fa-plus"></i> Tambah Data
      </button>
      <!-- Dropdown Filter -->
      <select v-model="kategoriFilter" @change="filterKategori(kategoriFilter)" class="filter-dropdown">
        <option value="">Show All</option>
        <option value="uang-masuk">Uang Masuk</option>
        <option value="uang-keluar">Uang Keluar</option>
      </select>
      
      <p class="saldo-keterangan">Saldo Anda: {{ formatRupiah(saldo) }}</p>
    </div>

    <!-- Modal untuk Uang Masuk -->
    <div v-if="showUangMasukModal" class="modal-overlay" @click.self="showUangMasukModal = false">
      <div class="modal-content">
        <button @click="showUangMasukModal = false" class="modal-close-button">
          <i class="fas fa-times"></i>
        </button>
        <h2>Tambah Uang Masuk</h2>
        <form @submit.prevent="submitUangMasuk">
          <div class="form-group">
            <label for="uang-masuk-jumlah">Jumlah Uang:</label>
            <input type="number" id="uang-masuk-jumlah" v-model="uangMasukForm.jumlahUang" required />
          </div>
          <div class="form-group">
            <label for="uang-masuk-keterangan">Keterangan:</label>
            <textarea id="uang-masuk-keterangan" v-model="uangMasukForm.keterangan" required></textarea>
          </div>
          <button type="submit" class="submit-button">
            <i class="fas fa-check"></i> Submit
          </button>
        </form>
      </div>
    </div>

    <!-- Modal untuk Tambah/Edit Data -->
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-content">
        <button @click="showModal = false" class="modal-close-button">
          <i class="fas fa-times"></i>
        </button>
        <h2>{{ editMode ? "Edit Data" : "Tambah Data" }}</h2>
        <form @submit.prevent="submitData">
          <div class="form-group">
            <label for="nomor-urut">Nomor Urut:</label>
            <input type="text" id="nomor-urut" v-model="form.nomorUrut" readonly />
          </div>
          <div class="form-group">
            <label for="tanggal">Tanggal:</label>
            <input type="date" id="tanggal" v-model="form.tanggal" required />
          </div>
          <div class="form-group">
            <label for="jumlah-uang">Jumlah Uang:</label>
            <input type="number" id="jumlah-uang" v-model="form.jumlahUang" required />
          </div>
          <div class="form-group">
            <label for="kategori">Kategori:</label>
            <select id="kategori" v-model="form.kategori" required>
              <option value="uang-masuk">Uang Masuk</option>
              <option value="uang-keluar">Uang Keluar</option>
            </select>
          </div>
          <div class="form-group">
            <label for="keterangan">Keterangan:</label>
            <textarea id="keterangan" v-model="form.keterangan" required></textarea>
          </div>
          <button type="submit" class="submit-button">
            <i class="fas fa-check"></i> {{ editMode ? "Update" : "Submit" }}
          </button>
        </form>
      </div>
    </div>

    <div v-if="dataList.length > 0" class="data-list">
      <h2>Data Keuangan</h2>
      <table>
        <thead>
          <tr>
            <th>Nomor Urut</th>
            <th>Tanggal</th>
            <th>Jumlah Uang</th>
            <th>Kategori</th>
            <th>Keterangan</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="data in filteredDataList" :key="data.nomorUrut">
            <td>{{ data.nomorUrut }}</td>
            <td>{{ data.tanggal }}</td>
            <td>{{ formatRupiah(data.jumlahUang) }}</td>
            <td>{{ data.kategori === 'uang-masuk' ? 'Uang Masuk' : 'Uang Keluar' }}</td>
            <td>{{ data.keterangan }}</td>
            <td>
              <button @click="editData(data)" class="edit-button">
                <i class="fas fa-edit"></i> Edit
              </button>
              <button @click="deleteData(data.nomorUrut)" class="delete-button">
                <i class="fas fa-trash"></i> Delete
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      showModal: false,
      showUangMasukModal: false,
      editMode: false,
      currentNomorUrut: 1,
      form: {
        nomorUrut: '',
        tanggal: '',
        jumlahUang: '',
        kategori: '',
        keterangan: ''
      },
      uangMasukForm: {
        jumlahUang: '',
        keterangan: ''
      },
      dataList: [],
      kategoriFilter: '', // Variable to store the current filter category
      saldo: 0 // Variable to store the current saldo
    };
  },
  computed: {
    filteredDataList() {
      if (this.kategoriFilter) {
        return this.dataList.filter(data => data.kategori === this.kategoriFilter);
      }
      return this.dataList;
    }
  },
  watch: {
    dataList: {
      handler() {
        this.updateSaldo();
        this.saveDataToLocalStorage();
      },
      deep: true
    }
  },
  created() {
    this.loadDataFromLocalStorage();
  },
  methods: {
    openModal() {
      this.form.nomorUrut = this.dataList.length > 0 ? Math.max(...this.dataList.map(data => data.nomorUrut)) + 1 : 1;
      this.showModal = true;
      this.editMode = false;
    },
    openUangMasukModal() {
      this.uangMasukForm.jumlahUang = '';
      this.uangMasukForm.keterangan = '';
      this.showUangMasukModal = true;
    },
    submitData() {
      if (this.editMode) {
        const index = this.dataList.findIndex(data => data.nomorUrut === this.form.nomorUrut);
        if (index !== -1) {
          this.dataList.splice(index, 1, { ...this.form });
        }
      } else {
        this.dataList.push({ ...this.form });
        this.sortDataList();
      }
      this.showModal = false;
      this.resetForm();
    },
    submitUangMasuk() {
      const newData = {
        nomorUrut: this.dataList.length > 0 ? Math.max(...this.dataList.map(data => data.nomorUrut)) + 1 : 1,
        tanggal: new Date().toISOString().split('T')[0],
        jumlahUang: this.uangMasukForm.jumlahUang,
        kategori: 'uang-masuk',
        keterangan: this.uangMasukForm.keterangan
      };
      this.dataList.push(newData);
      this.sortDataList();
      this.showUangMasukModal = false;
    },
    editData(data) {
      this.form = { ...data };
      this.showModal = true;
      this.editMode = true;
    },
    deleteData(nomorUrut) {
      this.dataList = this.dataList.filter(data => data.nomorUrut !== nomorUrut);
      this.sortDataList();
    },
    sortDataList() {
      this.dataList.sort((a, b) => a.nomorUrut - b.nomorUrut);
      this.dataList.forEach((data, index) => {
        data.nomorUrut = index + 1;
      });
      this.currentNomorUrut = this.dataList.length > 0 ? Math.max(...this.dataList.map(data => data.nomorUrut)) + 1 : 1;
    },
    filterKategori(kategori) {
      this.kategoriFilter = kategori;
    },
    updateSaldo() {
      this.saldo = this.dataList.reduce((acc, data) => {
        if (data.kategori === 'uang-masuk') {
          return acc + parseFloat(data.jumlahUang);
        } else if (data.kategori === 'uang-keluar') {
          return acc - parseFloat(data.jumlahUang);
        }
        return acc;
      }, 0);
    },
    resetForm() {
      this.form = {
        nomorUrut: '',
        tanggal: '',
        jumlahUang: '',
        kategori: '',
        keterangan: ''
      };
    },
    loadDataFromLocalStorage() {
      const savedData = JSON.parse(localStorage.getItem('dataList')) || [];
      this.dataList = savedData;
      this.currentNomorUrut = this.dataList.length > 0 ? Math.max(...this.dataList.map(data => data.nomorUrut)) + 1 : 1;
    },
    saveDataToLocalStorage() {
      localStorage.setItem('dataList', JSON.stringify(this.dataList));
    },
    formatRupiah(value) {
      if (value) {
        return `Rp. ${parseFloat(value).toLocaleString('id-ID', { minimumFractionDigits: 0 })}`;
      }
      return 'Rp. 0';
    }
  }
};
</script>

<style scoped>


.filter-dropdown {
  padding: 10px 15px;
  border: 2px solid #bb02b2;
  border-radius: 5px;
  background: linear-gradient(135deg, #209c1c, #feb47b);
  color: #090c0e;
  cursor: pointer;
  font-size: 16px;
  transition: border-color 0.3s ease, color 0.3s ease;
}

.filter-dropdown:hover {
  border-color: #00b351;
  color: #101111;
}

.filter-dropdown:focus {
  outline: none;
  border-color: #00b33c;
}

.filter-dropdown option {
  padding: 10px;
}

.content {
  background-color: blue;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.button-container {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}

.filter-button {
  padding: 10px 20px;
  background-color: teal;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.filter-button:hover {
  background-color: darkcyan;
}

.tambah-data-button {
  padding: 10px 20px;
  background-color: green;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.tambah-data-button:hover {
  background-color: darkgreen;
}

.saldo-keterangan {
  color: white;
  margin-left: auto;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  width: 400px;
  position: relative; /* Added to position the close button */
}

.modal-close-button {
  text-transform: capitalize;
  position: absolute;
  top: 5px;
  right: 5px;
  background-color: #00b33c;
  border: none;
  font-size: 15px;
  color: #080707;

  cursor: pointer;
}

.modal-close-button:hover {
  color: #c71919;

}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  color: #00b351; /* Warna hijau terang */
  
}


.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
  color: #000;
}

.submit-button {
  margin-top: 10px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background-color: green;
  color: white;
}

.submit-button:hover {
  background-color: darkgreen;
}
.data-list {
  margin-top: 20px;
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.data-list table {
  width: 100%;
  border-collapse: collapse;
}

.data-list th,
.data-list td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}
.data-list td {
  color: #0d0e0d; /* Green color for table data */
}

.data-list th {
  background-color: #f2f2f2;
}

.data-list tr:hover {
  background-color: #f1f1f1;
}

.edit-button,
.delete-button {
  padding: 5px 10px;
  margin-right: 5px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

.edit-button {
  background-color: orange;
  color: white;
}

.edit-button:hover {
  background-color: darkorange;
}

.delete-button {
  background-color: red;
  color: white;
}

.delete-button:hover {
  background-color: darkred;
}
.filter-button i,
.tambah-data-button i,
.submit-button i,
.modal-close-button i,
.edit-button i,
.delete-button i {
  margin-right: 5px;
}

.modal-close-button {
  font-size: 24px;
  background-color: transparent;
  border: none;
  color: #000;
}

.modal-close-button:hover {
  color: #c71919;
}


/* Style untuk tampilan ponsel seperti Oppo A54 */
@media (max-width: 768px) {
  .content {
    padding: 10px;
  }

  .button-container {
    flex-direction: column;
    align-items: flex-start;
  }
  .form-group label {
    color: #00b351; /* Pastikan warna tetap diterapkan */
  
  padding-bottom: 20px;
}
.data-list td {
  color: #0d0e0d; /* Green color for table data */
}


  .filter-button,
  .tambah-data-button {
    width: 100%;
    margin-bottom: 10px;
  }

  .saldo-keterangan {
    margin-left: 0;
    margin-top: 10px;
  }

  .modal-content {
    width: 90%;
  }

  .filter-dropdown {
    width: 100%;
  }
  .submit-button{
    margin-right: 5px;
  }

  .data-list table {
    font-size: 14px;
  }

  .data-list td {
  color: #0d0e0d; /* Green color for table data */
}

  .edit-button,
  .delete-button {
    padding: 8px 12px;
    font-size: 14px;
  }

  .modal-close-button {
    font-size: 20px;
  }
}

</style>
