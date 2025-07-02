| Nama | Silfa Salsa Bila Putri |
| --- | --- |
| NIM | 312310607 |
| Kelas | TI.23.A6 |
| Mata Kuliah | Pemrograman Web 2 |

# PRAKTIKUM 6

-  Memodifikasi Controller untuk Menyimpan Gambar

   Update method add() Artikel.php menjadi:

   ```
   public function add()
   {
       // validasi data
       $validation = \Config\Services::validation();
       $validation->setRules(['judul' => 'required']);
       $isDataValid = $validation->withRequest($this->request)->run();

       if ($isDataValid)
       {
           $file = $this->request->getFile('gambar');
           $file->move(ROOTPATH . 'public/gambar'); // Pindahkan file ke folder public/gambar

           $artikel = new ArtikelModel();
          $artikel->insert([
               'judul' => $this->request->getPost('judul'),
               'isi' => $this->request->getPost('isi'),
               'slug' => url_title($this->request->getPost('judul')),
               'gambar' => $file->getName(), // Simpan nama file ke database
           ]);

           return redirect('admin/artikel');
       }

       $title = "Tambah Artikel";
       return view('artikel/form_add', compact('title'));
   }
   ```

   getFile('gambar') mengambil file yang dikirim dari form. move() memindahkan file ke folder /public/gambar. getName() digunakan untuk menyimpan nama file ke kolom gambar di tabel.

- Memodifikasi Form Tambah Artikel

  Ubah tag ``` <form> ``` agar bisa mengunggah file:

  ``` <form action="" method="post" enctype="multipart/form-data"> ```

  Tambahkan input untuk file gambar:

  ```
  <p>
      <input type="file" name="gambar">
  </p>
  ```

  enctype="multipart/form-data" wajib untuk upload file. <input type="file" name="gambar"> membuat field unggah file.

- Menyimpan Gambar ke Folder public/gambar

  Langkah yang terjadi saat submit: File dikirim dari browser, Disimpan otomatis ke folder public/gambar/ berkat fungsi move(), Nama file disimpan ke kolom gambar di database artikel.

- Menampilkan Gambar di Daftar dan Detail Artikel

  ``` <img src="<?= base_url('/gambar/' . $row['gambar']); ?>" alt="<?= $row['judul']; ?>"> ```

  Gambar ditampilkan menggunakan path /gambar/nama_file, File sebelumnya sudah disimpan di folder public/gambar, jadi aman diakses lewat base_url().

HASIL OUTPUT
---

![Screenshot (42)](https://github.com/user-attachments/assets/85f6fa11-f814-4e39-9f2a-d8bacf1f7ee7)

# PRAKTIKUM 8 

- Persiapan Awal

  Saya membuka kembali project CodeIgniter yang sudah pernah dibuat sebelumnya. Project ini sudah memiliki model ArtikelModel yang sebelumnya digunakan untuk CRUD artikel secara biasa (tanpa AJAX).

- Menambahkan jQuery ke Proyek

  Agar bisa menggunakan fitur AJAX dengan lebih praktis, saya mengunduh jQuery versi 3.6.0 dari situs resminya https://jquery.com, lalu menyimpan file jquery-3.6.0.min.js ke dalam folder public/assets/js. Tujuannya adalah agar bisa menyisipkan script jQuery dari file lokal, tanpa harus mengandalkan CDN.

- Membuat Controller AjaxController.php

  Saya membuat controller baru dengan nama AjaxController.php di dalam folder app/Controllers. Fungsinya ada dua:
  
  - getData(): untuk mengambil semua data artikel dan mengirimnya dalam format JSON.
  - delete($id): untuk menghapus artikel berdasarkan ID, lalu mengembalikan statusnya dalam JSON juga.
  
  Cuplikan fungsinya:

  ```
  public function getData() {
      $model = new ArtikelModel();
      $data = $model->findAll();
      return $this->response->setJSON($data);
  }
  ```

- Membuat Halaman View AJAX

  Saya membuat file view ajax/index.php untuk menampilkan daftar artikel secara dinamis ke dalam tabel HTML. Di dalam view ini:

  - Saya menyisipkan file jquery-3.6.0.min.js yang tadi sudah diunduh.
  - Saya membuat tabel kosong, lalu mengisi barisnya menggunakan AJAX.
  - Data diambil dari ajax/getData dan ditampilkan menggunakan JavaScript.

  Cuplikan kode jQuery:

  ```
  $.ajax({
    url: "<?= base_url('ajax/getData') ?>",
    method: "GET",
    dataType: "json",
    success: function(data) {
        // Tampilkan datanya ke tabel
    }
  });
  ```

- Menambahkan Indikator Loading

  Sebelum data berhasil diambil dari server, saya menambahkan baris sementara di tabel dengan tulisan "Loading data..." untuk memberi tahu user bahwa proses sedang berlangsung.

- Menampilkan Data ke Tabel

  Begitu data diterima dari server, saya melakukan looping terhadap array artikel, lalu menyusunnya dalam bentuk baris HTML (<tr>), dan disisipkan ke dalam tabel.

- Menambahkan Tombol Aksi (Edit dan Delete)

  Setiap baris artikel di tabel memiliki dua tombol:
  - Edit: Mengarah ke halaman artikel/edit/{id} seperti biasa.
  - Delete: Akan menjalankan fungsi AJAX DELETE untuk menghapus data langsung dari tampilan.
 
  Cuplikan HTML tombol:

  ```
  <a href="<?= base_url('artikel/edit/') ?>' + row.id" class="btn btn-primary">Edit</a>
  <a href="#" class="btn btn-danger btn-delete" data-id="' + row.id + '">Delete</a>
  ```

- Fungsi Hapus Data via AJAX
  
  Ketika tombol Delete diklik, akan muncul konfirmasi dulu. Jika user setuju, maka akan dikirimkan permintaan DELETE ke endpoint artikel/delete/{id}. Setelah berhasil, data akan dimuat ulang agar perubahan langsung terlihat.
  Cuplikan kode hapus:

  ```
  $.ajax({
    url: "<?= base_url('artikel/delete/') ?>" + id,
    method: "DELETE",
    success: function(data) {
      loadData(); // Refresh data
    }
  });
  ```

# PRAKTIKUM 9

- Persiapan

  Sebelum memulai, saya memastikan:
  - MySQL Server dalam keadaan aktif.
  - Database lab_ci4 telah berisi tabel artikel dan kategori yang sudah terisi data.
  - Library jQuery versi 3.6.0 sudah tersedia, bisa dari file lokal atau via CDN.
 
- Modifikasi admin_index() di Controller Artikel

  Saya mengubah method admin_index() agar bisa mendeteksi request AJAX. Kalau request berasal dari AJAX, maka response-nya dikirim dalam bentuk JSON (bukan HTML).
  Perubahan penting di method ini:

  ```
  if ($this->request->isAJAX()) {
      return $this->response->setJSON($data);
  }
  ```

  Penjelasan variabel dan fungsi:
  - $page = $this->request->getVar('page') ?? 1: mengambil nomor halaman saat ini.
  - ->paginate(10, 'default', $page): membatasi hanya 10 data artikel per halaman.
  - ->join('kategori', ...): menggabungkan tabel artikel dengan kategori.
  - JSON akan digunakan oleh JavaScript untuk mengisi tabel di view nanti.

- Modifikasi Tampilan admin_index.php

  Saya membersihkan isi view lama yang masih menampilkan data artikel secara langsung. Sebagai gantinya, saya menyiapkan dua container kosong:

  ```
  <div id="article-container"></div>
  <div id="pagination-container"></div>
  ```

  Semua data di dalam ini akan di-render oleh jQuery berdasarkan response dari AJAX.

- Menambahkan Form Pencarian dan Filter Kategori

  Saya tetap menggunakan form seperti sebelumnya untuk input kata kunci pencarian dan memilih kategori. Tapi kali ini form tidak mengirimkan data secara konvensional (reload), melainkan diproses menggunakan JavaScript saat tombol submit ditekan.
  Cuplikan form:

  ```
  <form id="search-form" class="form-inline">
    <input type="text" name="q" id="search-box">
    <select name="kategori_id" id="category-filter"> ... </select>
    <input type="submit" value="Cari">
  </form>
  ```

- AJAX – Mengambil dan Menampilkan Data Tanpa Reload

  Saya menambahkan script jQuery untuk:
  - Mengambil data artikel via endpoint /admin/artikel (dengan parameter q, kategori_id, dan page).
  - Memproses dan menampilkan hasilnya di dalam <div id="article-container">.
  - Menampilkan tombol pagination di <div id="pagination-container">.
  Cuplikan fungsi utama:

  ```
  const fetchData = (url) => {
    $.ajax({
      url: url,
      type: 'GET',
      dataType: 'json',
      headers: { 'X-Requested-With': 'XMLHttpRequest' },
      success: function(data) {
        renderArticles(data.artikel)
        renderPagination(data.pager, data.q, data.kategori_id);
      }
    });
  }
  ```

- Fungsi renderArticles() dan renderPagination()

  Untuk menampilkan data yang didapat, saya membuat dua fungsi:
  - renderArticles(): menampilkan data ke dalam tabel.
  - renderPagination(): menampilkan navigasi pagination berdasarkan pager.links.
  Pagination juga mendukung pencarian dan filter kategori, karena q dan kategori_id ikut dikirim lewat URL.  Pagination tetap bisa diklik karena saya membungkusnya dalam <ul class="pagination"> dan menggunakan href dinamis.

- Event Listener pada Form dan Filter

  Saya menambahkan event listener untuk Tombol submit form pencarian (agar mengambil data via AJAX), dan Dropdown kategori (agar langsung refresh data jika dipilih).
  Contoh event:

  ```
  searchForm.on('submit', function(e) {
    e.preventDefault();
    const q = searchBox.val();
    const kategori_id = categoryFilter.val();
    fetchData(`/admin/artikel?q=${q}&kategori_id=${kategori_id}`);
    });
  ```

# PRAKTIKUM 10

- Persiapan

  Sebelum memulai coding, saya melakukan beberapa langkah awal:
  - Menjalankan MySQL Server.
  - Memastikan database lab_ci4 sudah tersedia dan tabel artikel sudah berisi data.
  - Menginstal aplikasi Postman dari https://www.postman.com/downloads untuk keperluan pengujian REST API.

- Membuat REST API Controller

  Saya membuat file baru bernama Post.php di dalam folder app/Controllers. File ini menggunakan controller khusus bawaan CodeIgniter, yaitu ResourceController, yang memudahkan dalam membuat API standar.
  Isi file Post.php:

  ```
  namespace App\Controllers;
  use CodeIgniter\RESTful\ResourceController;
  use CodeIgniter\API\ResponseTrait;
  use App\Models\ArtikelModel;

  class Post extends ResourceController
  {
      use ResponseTrait;

      public function index()
      {
          $model = new ArtikelModel();
          $data['artikel'] = $model->orderBy('id', 'DESC')->findAll();
          return $this->respond($data);
      }

      public function create()
      {
          $model = new ArtikelModel();
         $data = [
              'judul' => $this->request->getVar('judul'),
              'isi' => $this->request->getVar('isi'),
          ];
          $model->insert($data)
          return $this->respondCreated([
              'status' => 201,
                'messages' => ['success' => 'Data artikel berhasil ditambahkan.']
          ]);
    }

      public function show($id = null)
        {
          $model = new ArtikelModel();
          $data = $model->where('id', $id)->first();
          return $data ? $this->respond($data) : $this->failNotFound('Data tidak ditemukan.');
      }

      public function update($id = null)
      {
          $model = new ArtikelModel();
          $data = [
              'judul' => $this->request->getVar('judul'),
              'isi' => $this->request->getVar('isi'),
          ];
          $model->update($id, $data);
          return $this->respond([
              'status' => 200,
              'messages' => ['success' => 'Data artikel berhasil diubah.']
          ]);
      }

      public function delete($id = null)
      {
          $model = new ArtikelModel();
          $data = $model->find($id);
          if ($data) {
              $model->delete($id);
              return $this->respondDeleted([
                  'status' => 200,
                  'messages' => ['success' => 'Data artikel berhasil dihapus.']
                ]);
          }
          return $this->failNotFound('Data tidak ditemukan.');
      }
  }
  ```

  Fungsi-fungsi di atas menjawab kebutuhan HTTP:
  - GET /post → Ambil semua artikel.
  - GET /post/2 → Ambil artikel dengan ID 2.
  - POST /post → Tambah data artikel.
  - PUT /post/2 → Ubah artikel dengan ID 2.
  - DELETE /post/2 → Hapus artikel dengan ID 2.
  
- Menambahkan Routing untuk REST API

  Saya membuka file app/Config/Routes.php, lalu menambahkan route baru:

  ``` $routes->resource('post'); ```

  Dengan hanya satu baris ini, CodeIgniter otomatis menghasilkan semua endpoint RESTful sesuai standar HTTP (GET, POST, PUT, DELETE).
  Untuk melihat semua endpoint yang tersedia, saya jalankan perintah:

  ``` php spark routes ```

- Pengujian API Menggunakan Postman

  Saya menguji setiap method dengan Postman:
  1. Menampilkan Semua Data (GET)
     - Method: GET
     - URL: http://localhost:8080/post
     - ✅ Hasil: Semua data artikel ditampilkan dalam format JSON.
  2. Menampilkan Data Spesifik (GET by ID)
     - Method: GET
     - URL: http://localhost:8080/post/2
     - ✅ Hasil: Menampilkan data artikel dengan ID 2.
  3. Menambahkan Artikel (POST)
     - Method: POST
     - URL: http://localhost:8080/post
     - Body (x-www-form-urlencoded):
       - judul: Artikel Baru dari API
       - isi: Ini adalah isi artikel yang ditambahkan via Postman
     - ✅ Hasil: Pesan berhasil ditambahkan (status 201).
  4. Mengubah Artikel (PUT)
     - Method: PUT
     - URL: http://localhost:8080/post/2
     - Body (x-www-form-urlencoded):
       - judul: Judul Baru
       - isi: Isi artikel sudah diperbarui
     - ✅ Hasil: Pesan berhasil diubah (status 200).
  5. Menghapus Artikel (DELETE)
     - Method: DELETE
     - URL: http://localhost:8080/post/3
     - ✅ Hasil: Pesan sukses dihapus jika ID ada, atau error jika ID tidak ditemukan.

# PRAKTIKUM 11

- Persiapan dan Struktur Project
  Sebelum mulai, saya menyiapkan struktur folder sederhana sebagai berikut:

  ```
  project/
  ├── index.html
  └── assets/
      ├── css/
      │   └── style.css
      └── js/
          └── app.js
  ```

  Kemudian saya menggunakan CDN Vue.js dan Axios agar tidak perlu install NPM.

  ```
  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  ```

- Membuat Halaman index.html
  Isi file index.html menampilkan:
  - Judul daftar artikel.
  - Tabel artikel.
  - Tombol tambah.
  - Form input (modal) untuk tambah/ubah.
  - Tautan ke Vue dan Axios (CDN) serta file app.js.
  Contoh bagian inti HTML:

  ```
    <div id="app">
    <h1>Daftar Artikel</h1>
    <button id="btn-tambah" @click="tambah">Tambah Data</button>

    <!-- Modal Form -->
    <div class="modal" v-if="showForm">
      <div class="modal-content">
        <span class="close" @click="showForm = false">&times;</span>
        <form id="form-data" @submit.prevent="saveData">
            <h3>{{ formTitle }}</h3>
            <input type="text" v-model="formData.judul" placeholder="Judul" required />
            <textarea rows="10" v-model="formData.isi"></textarea>
          <select v-model="formData.status">
            <option v-for="option in statusOptions" :value="option.value">{{ option.text }}</option>
            </select>
          <button type="submit">Simpan</button>
          <button @click="showForm = false">Batal</button>
        </form>
      </div>
    </div>

    <!-- Tabel Artikel -->
    <table>
      <thead>
        <tr>
          <th>ID</th><th>Judul</th><th>Status</th><th>Aksi</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in artikel">
          <td>{{ row.id }}</td>
          <td>{{ row.judul }}</td>
          <td>{{ statusText(row.status) }}</td>
          <td>
            <a href="#" @click="edit(row)">Edit</a>
            <a href="#" @click="hapus(index, row.id)">Hapus</a>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  ```

![Screenshot (46)](https://github.com/user-attachments/assets/78a8e9df-c58b-4ee8-a3a3-1c03f87a082b)

- Script Vue.js (assets/js/app.js)
  Di file ini, saya mendefinisikan:
  - data() → menyimpan state artikel, form, dan modal.
  - mounted() → menjalankan loadData() saat halaman dimuat.
  - methods:
    - loadData() → ambil semua artikel dari endpoint /post.
    - tambah() → menampilkan form kosong.
    - edit(data) → mengisi form dengan data yang mau diubah.
    - hapus(index, id) → menghapus artikel.
    - saveData() → menyimpan data baru atau update data.
    - statusText(status) → ubah angka jadi teks (Publish / Draft).
  Contoh kode:

  ```
  const { createApp } = Vue
  const apiUrl = 'http://localhost/labci4/public'

  createApp({
    data() {
      return {
        artikel: [],
        formData: { id: null, judul: '', isi: '', status: 0 },
        showForm: false,
        formTitle: 'Tambah Data',
        statusOptions: [
          { text: 'Draft', value: 0 },
          { text: 'Publish', value: 1 }
        ]
      }
    },
    mounted() {
      this.loadData()
    },
    methods: {
      loadData() {
        axios.get(`${apiUrl}/post`)
          .then(res => { this.artikel = res.data.artikel })
          .catch(err => console.log(err))
      },
      tambah() {
        this.formTitle = 'Tambah Data'
        this.formData = { id: null, judul: '', isi: '', status: 0 }
        this.showForm = true
      },
      edit(data) {
        this.formTitle = 'Ubah Data'
        this.formData = { ...data }
        this.showForm = true
      },
      hapus(index, id) {
        if (confirm('Yakin hapus data?')) {
          axios.delete(`${apiUrl}/post/${id}`)
            .then(() => this.artikel.splice(index, 1))
            .catch(err => console.log(err))
        }
      },
      saveData() {
        const method = this.formData.id ? 'put' : 'post'
        const url = this.formData.id
          ? `${apiUrl}/post/${this.formData.id}`
          : `${apiUrl}/post`
        axios[method](url, this.formData)
          .then(() => this.loadData())
          .catch(err => console.log(err))
        this.showForm = false
      },
      statusText(status) {
        return status == 1 ? 'Publish' : 'Draft'
      }
    }
  }).mount('#app')
  ```

- File CSS (assets/css/style.css)
  Saya menggunakan file style.css untuk styling layout form, tombol, dan tabel.
  Contoh:

  ```
  table { width: 100%; border-collapse: collapse; }
  th, td { padding: 10px; border: 1px solid #ddd; }
  th { background-color: #5778ff; color: white; }
  .modal { position: fixed; background: rgba(0,0,0,0.4); ... }
  ```

![Screenshot (47)](https://github.com/user-attachments/assets/39f96638-3bf1-45e1-be7e-969aae175361)

