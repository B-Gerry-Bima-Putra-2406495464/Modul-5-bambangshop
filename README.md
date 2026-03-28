# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. Pada kasus BambangShop ini, penggunaan interface/trait tidak terlalu diwajibkan dan single Model struct Subscriber sudah cukup. Hal ini karena sifat subscriber di sini sangat seragam, yaitu hanya menerima data via HTTP POST request ke URL masing-masing. Kita tidak memiliki berbagai tipe subscriber dengan implementasi internal logic yang berbeda-beda di dalam aplikasi Publisher.

2. Menggunakan DashMap sangat disarankan dan diperlukan karena kita butuh keunikan berdasarkan URL. Jika menggunakan Vec (list), kita harus melakukan iterasi dengan kompleksitas waktu O(n) untuk mencari subscriber yang ingin dihapus atau dicek. Dengan DashMap, pencarian, penambahan, dan penghapusan berdasarkan key (URL) bisa dilakukan jauh lebih cepat dengan kompleksitas O(1).

3. Ya, kita tetap membutuhkan DashMap. Singleton hanya menjamin bahwa database in-memory tersebut diciptakan tepat satu kali (single instance) secara global. Namun, Singleton tidak otomatis menangani isu thread-safety. DashMap lah yang bekerja menangani penguncian (locking) saat banyak thread di web server mencoba menambah atau menghapus subscriber secara bersamaan agar tidak terjadi data race.

#### Reflection Publisher-2
1. Memisahkan Service dan Repository menerapkan prinsip Single Responsibility Principle (SRP) dan Separation of Concerns. Model murni untuk mendefinisikan struktur data, Repository khusus menangani logika akses/penyimpanan data (seperti ke database atau memori), dan Service khusus menangani business logic atau aturan bisnis. Pemisahan ini membuat kode lebih rapi, modular, dan lebih mudah di-test (unit testing) secara independen.

2. Jika semuanya ditaruh di Model, file Model akan memiliki terlalu banyak fungsi di satu tempat. Interaksi antar model (seperti Program, Subscriber, Notification) akan menciptakan ketergantungan yang kuat. Akibatnya, mengubah satu logika kecil pada notifikasi bisa berdampak langsung atau merusak kode yang mengurus subscriber atau product.

3. Postman sangat membantu karena kita bisa langsung melakukan HTTP Request (POST, GET) ke API endpoint tanpa harus membuat aplikasi frontend terlebih dahulu. Fitur yang menarik adalah "Collections" yang bisa menyimpan skenario tes berulang kali dan pengiriman JSON body raw yang sangat praktis.

#### Reflection Publisher-3
1. Pada tutorial ini, kita menggunakan variasi Push model. Publisher (Main App) secara aktif mendorong (pushes) data notifikasi (payload berisi detail produk dan status) langsung ke setiap Subscriber yang terdaftar begitu sebuah event terjadi.

2. Jika kita menggunakan Pull model, keuntungannya adalah subscriber memegang kendali penuh kapan mereka ingin mengambil data, sehingga mereka tidak akan kewalahan (overloaded) jika trafik sedang tinggi. Namun, kerugiannya adalah subscriber harus melakukan polling (mengecek secara berkala) ke Publisher. Ini membuang-buang sumber daya resource jaringan dan komputasi jika ternyata tidak ada produk baru yang ditambahkan/diubah.

3. Jika kita tidak menggunakan multi-threading, proses notifikasi akan berjalan secara sinkron (synchronous). Artinya, jika ada 100 subscriber, Publisher harus menunggu HTTP request selesai dikirim ke subscriber pertama, baru lanjut ke yang kedua, dan seterusnya. Jika ada satu URL subscriber yang down atau lambat merespons (timeout), keseluruhan proses (termasuk respons aplikasi utama saat create/delete produk) akan terhenti atau melambat drastis.
