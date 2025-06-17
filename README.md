# 🤖 Tugas 3 - Pemrograman Jaringan (C) - 5025221034

Repositori ini berisi tugas 3 yang dibuat sebagai bagian dari tugas mata kuliah Praktikum Pemrograman Jaringan (Kelas C).

Disusun oleh:

Nama: Bintang Wibi Hanoraga

NRP: 5025221034

---

### Persoalan 1
- Unggah Berkas
#### Berkas yang diunggah harus dienkode terlebih dahulu ke dalam format Base64 sebelum dikirimkan.
- Hapus Berkas
#### Terapkan kapabilitas untuk menyingkirkan berkas yang sudah ada di peladen.
---
### Penambahan PROTOKOL.TXT
```
UPLOAD
* TUJUAN: Mengirim berkas dari klien ke peladen.
* PARAMETER:
  - PARAMETER1: nama berkas
  - PARAMETER2: isi berkas dalam base64
* RESULT:
- BERHASIL:
  - status: OK
  - data: pesan keberhasilan
- GAGAL:
  - status: ERROR
  - data: pesan kekeliruan

DELETE
* TUJUAN: Menghapus berkas di peladen berdasarkan nama berkas.
* PARAMETER:
  - PARAMETER1: nama berkas
* RESULT:
- BERHASIL:
  - status: OK
  - data: pesan keberhasilan
- GAGAL:
  - status: ERROR
  - data: pesan kekeliruan
```

---
### Integrasikan fungsi `upload` dan `delete` di `file_interface.py`
- Modifikasi berkas `file_interface.py` dan sisipkan dua metode baru: `upload` dan `delete`.
- Tambahkan ini di dalam kelas `FileInterface`:
```
    def upload(self, params=[]):
        try:
            filename = params[0]
            filedata_base64 = params[1]
            filedata = base64.b64decode(filedata_base64)

            with open(filename, 'wb') as f:
                f.write(filedata)

            return dict(status='OK', data=f'File {filename} berhasil diupload')
        except Exception as e:
            return dict(status='ERROR', data=str(e))

    def delete(self, params=[]):
        try:
            filename = params[0]
            if not os.path.exists(filename):
                return dict(status='ERROR', data=f'File {filename} tidak ditemukan')

            os.remove(filename)
            return dict(status='OK', data=f'File {filename} berhasil dihapus')
        except Exception as e:
            return dict(status='ERROR', data=str(e))

```
Fungsi `upload(self, params=[])` dimanfaatkan untuk mengelola proses pengiriman berkas dari klien ke peladen. Fungsi ini menerima masukan berupa daftar `params`, di mana `params[0]` adalah nama berkas (`filename`) dan `params[1]` adalah data berkas yang telah dikodekan dalam format **Base64** (`filedata_base64`). Data yang telah dikodekan ini kemudian diuraikan kembali ke wujud aslinya menggunakan `base64.b64decode()`, dan hasilnya disimpan ke dalam berkas baru dengan nama `filename` menggunakan mode penulisan biner (`'wb'`). Jika operasi pengunggahan sukses, fungsi akan mengembalikan objek kamus (`dictionary`) dengan status bernilai **'OK'** dan notifikasi bahwa berkas berhasil diunggah. Apabila terjadi kegagalan selama proses, fungsi akan menangkap pengecualian (`exception`) dan mengembalikan objek kamus dengan status bernilai **'ERROR'** beserta rincian kesalahan.

Fungsi `delete(self, params=[])` bertugas untuk menghapus berkas yang telah tersimpan di peladen. Fungsi ini juga menerima masukan berupa daftar `params`, di mana `params[0]` memuat nama berkas yang ingin disingkirkan. Fungsi ini terlebih dahulu memeriksa eksistensi berkas tersebut di sistem menggunakan `os.path.exists()`. Jika berkas tidak ditemukan, maka fungsi akan mengembalikan objek kamus dengan status bernilai **'ERROR'** dan notifikasi bahwa berkas tidak ditemukan. Namun, jika berkas ditemukan, maka berkas akan dihapus menggunakan `os.remove()`. Setelah penghapusan berhasil, fungsi akan mengembalikan objek kamus dengan status bernilai **'OK'** dan notifikasi bahwa berkas berhasil dihapus. Jika terjadi kegagalan selama proses, fungsi akan menangkap pengecualian (`exception`) dan mengembalikan objek kamus dengan status bernilai **'ERROR'** beserta detail pesan kesalahannya.

---
### Perbarui `file_protocol.py` agar mengenali perintah `UPLOAD` dan `DELETE`
- Karena di `file_protocol.py` kita sudah menggunakan `getattr(self.file, c_request)` untuk memanggil fungsi berdasarkan nama perintah, maka kita tidak perlu mengubah banyak.
- Akan tetapi, kita perlu memastikan bahwa saat perintah **UPLOAD** diproses, parameter-nya tepat—karena ia memerlukan dua parameter: nama berkas dan data **Base64** berkas.
```
class FileProtocol:
    def __init__(self):
        self.file = FileInterface()
    def proses_string(self,string_datamasuk=''):
        logging.warning(f"string diproses: {string_datamasuk}")
        c = shlex.split(string_datamasuk)
        try:
            c_request = c[0].strip().lower()
            logging.warning(f"memproses request: {c_request}")
            params = [x for x in c[1:]]
            cl = getattr(self.file, c_request)(params)
            return json.dumps(cl)
        except Exception as e:
            return json.dumps(dict(status='ERROR', data='request tidak dikenali'))
```
Kelas `FileProtocol` digunakan sebagai tata cara penghubung antara instruksi dalam wujud teks dan pemanggilan fungsi yang selaras dalam antarmuka berkas (`FileInterface`). Pada saat inisiasi (`__init__`), kelas ini menciptakan objek `self.file` yang merupakan instans dari kelas `FileInterface`, tempat fungsi-fungsi seperti `upload`, `delete`, dan lain-lain berada.

Fungsi esensial dalam kelas ini adalah `proses_string(self, string_datamasuk='')`, yang bertugas mengolah masukan berupa rentetan karakter perintah. Rentetan karakter tersebut terlebih dahulu dipisahkan menjadi bagian-bagian menggunakan `shlex.split()`, yang berguna agar rentetan karakter dengan spasi yang terdapat dalam tanda kutip tetap dianggap sebagai satu kesatuan. Bagian pertama (`c[0]`) dianggap sebagai nama metode (misalnya `upload` atau `delete`), lalu diubah menjadi huruf kecil dan dibersihkan dari spasi menggunakan `strip().lower()`. Bagian-bagian berikutnya (`c[1:]`) dijadikan sebagai parameter (`params`) yang akan diteruskan ke fungsi yang bersesuaian.

Fungsi kemudian memanfaatkan `getattr(self.file, c_request)` untuk mengaktifkan metode yang serasi dari objek `FileInterface` berdasarkan nama yang diberikan. Metode tersebut kemudian dieksekusi dengan `params` sebagai argumen. Hasil dari eksekusi akan dikembalikan dalam format **JSON** menggunakan `json.dumps()`. Apabila terjadi kekeliruan — contohnya perintah tidak dikenali atau tidak sinkron dengan metode yang tersedia di `FileInterface` — maka fungsi akan mengembalikan respons **JSON** dengan status **'ERROR'** dan pesan 'request tidak dikenali'.

---

### Tambahkan fungsi `remote_upload()` dan `remote_delete()` di `file_client_cli.py`
- Kita akan menyisipkan dua kapabilitas baru di sisi klien:
`remote_upload(namafile)` dan `remote_delete(namafile)`

- Kedua kapabilitas ini akan mengirimkan instruksi ke peladen sesuai format protokol:
`UPLOAD <namafile> <base64_data>` dan `DELETE <namafile>`

```
def remote_upload(namafile=""):
    try:
        # baca file dan encode base64
        with open(namafile, "rb") as f:
            file_data = base64.b64encode(f.read()).decode()

        # kirim perintah ke server
        command_str = f"UPLOAD {namafile} {file_data}"
        hasil = send_command(command_str)

        if hasil['status'] == 'OK':
            print(f"File {namafile} berhasil diupload.")
        else:
            print("Gagal upload:", hasil['data'])
    except Exception as e:
        print("Terjadi kesalahan:", str(e))


def remote_delete(namafile=""):
    command_str = f"DELETE {namafile}"
    hasil = send_command(command_str)

    if hasil['status'] == 'OK':
        print(f"File {namafile} berhasil dihapus.")
    else:
        print("Gagal menghapus:", hasil['data'])
```
Fungsi `remote_upload(namafile="")` dipakai oleh klien untuk memuat berkas ke peladen. Fungsi ini mula-mula membuka berkas lokal berdasarkan nama yang disajikan melalui parameter `namafile`, membaca semua isinya dalam mode **biner** (`'rb'`), lalu mengkodekan isi berkas tersebut ke dalam format **Base64** menggunakan `base64.b64encode()`. Hasil pengodean tersebut diubah menjadi rentetan karakter **UTF-8** agar dapat dikirim melalui jaringan. Setelah itu, fungsi menyusun instruksi dalam bentuk rentetan karakter, diawali dengan kata kunci **UPLOAD**, diikuti dengan nama berkas dan isi berkas yang telah dikodekan, lalu dikirim ke peladen melalui fungsi `send_command()`. Jika peladen membalas dengan status **'OK'**, maka akan ditampilkan notifikasi bahwa berkas berhasil dimuat. Jika tidak, notifikasi kesalahan akan ditampilkan. Fungsi juga dilengkapi blok `try-except` untuk menangkap dan menampilkan notifikasi jika terjadi kekeliruan seperti berkas tidak ditemukan atau gagal dibaca.

Fungsi `remote_delete(namafile="")` dipergunakan untuk menyingkirkan berkas yang sebelumnya telah dimuat ke peladen. Fungsi ini menyusun instruksi dalam bentuk rentetan karakter dengan format **`DELETE <namafile>`** lalu mengirimkannya ke peladen melalui fungsi `send_command()`. Hasil balasan dari peladen kemudian diinspeksi. Jika status yang dikembalikan adalah **'OK'**, maka akan ditampilkan notifikasi bahwa berkas berhasil disingkirkan. Namun jika peladen mengembalikan status galat, maka notifikasi kegagalan akan ditampilkan. Fungsi ini lebih sederhana karena tidak mengolah isi berkas, melainkan hanya mengirimkan instruksi berdasarkan nama berkas yang ingin disingkirkan.

---

### Perbarui segmen `if __name__ == '__main__':`
```
if __name__=='__main__':
    server_address=('172.16.16.101',6969)

    remote_list()
    remote_get('donalbebek.jpg') # contoh berkas yang akan diminta ke peladen
    remote_upload('test.txt') # untuk pengujian unggah berkas ke peladen
    remote_delete('donalbebek.jpg')
    remote_list()

```
