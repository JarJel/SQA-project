# Use Case Diagram

```mermaid
useCaseDiagram
    actor "Customer" as C
    actor "Admin" as A

    package "Sistem Pemesanan Café" {
        usecase "Register/Login" as UC1
        usecase "Lihat Menu" as UC2
        usecase "Tambah ke Keranjang" as UC3
        usecase "Checkout & Pilih Tipe" as UC4
        usecase "Bayar (Cashless)" as UC5
        usecase "Kelola Menu (CRUD)" as UC6
        usecase "Update Status Pesanan" as UC7
    }

    C --> UC1
    C --> UC2
    C --> UC3
    C --> UC4
    C --> UC5
    
    A --> UC1
    A --> UC6
    A --> UC7
