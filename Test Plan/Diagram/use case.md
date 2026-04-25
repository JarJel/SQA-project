# Use Case Diagram - Sistem Pemesanan Café

Diagram ini mendefinisikan interaksi antara aktor (Customer & Admin) dengan fungsionalitas sistem.

```mermaid
graph LR
    %% Definisi Aktor
    Customer((Customer))
    Admin((Admin))

    subgraph "Sistem Pemesanan Café"
        UC1(Register / Login)
        UC2(Lihat Menu)
        UC3(Tambah ke Keranjang)
        UC4(Checkout & Pilih Tipe)
        UC5(Bayar Cashless)
        UC6(Kelola Menu CRUD)
        UC7(Update Status Pesanan)
    end

    %% Relasi Customer
    Customer --- UC1
    Customer --- UC2
    Customer --- UC3
    Customer --- UC4
    Customer --- UC5

    %% Relasi Admin
    Admin --- UC1
    Admin --- UC6
    Admin --- UC7
    
    %% Styling agar lebih rapi
    style Customer fill:#f9f,stroke:#333,stroke-width:2px
    style Admin fill:#bbf,stroke:#333,stroke-width:2px
    style UC1 fill:#fff,stroke:#333
    style UC2 fill:#fff,stroke:#333
    style UC3 fill:#fff,stroke:#333
    style UC4 fill:#fff,stroke:#333
    style UC5 fill:#fff,stroke:#333
    style UC6 fill:#fff,stroke:#333
    style UC7 fill:#fff,stroke:#333
