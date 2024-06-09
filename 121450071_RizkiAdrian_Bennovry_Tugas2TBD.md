---
jupyter:
  colab:
  kernelspec:
    display_name: Python 3
    name: python3
  language_info:
    name: python
  nbformat: 4
  nbformat_minor: 0
---

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="PuSEjbt6pY7Y" outputId="a58badb9-ab22-4346-e57d-d6c71044767b"}
``` python
import numpy as np
import pickle
from pathlib import Path

# Path to the unzipped CIFAR data
data_dir = Path("data/cifar-10-batches-py/")

# Unpickle function provided by the CIFAR hosts
def unpickle(file):
    with open(file, "rb") as fo:
        dict = pickle.load(fo, encoding="bytes")
    return dict

images, labels = [], []
for batch in data_dir.glob("data_batch_*"):
    batch_data = unpickle(batch)
    for i, flat_im in enumerate(batch_data[b"data"]):
        im_channels = []
        # Each image is flattened, with channels in order of R, G, B
        for j in range(3):
            im_channels.append(
                flat_im[j * 1024 : (j + 1) * 1024].reshape((32, 32))
            )
        # Reconstruct the original image
        images.append(np.dstack((im_channels)))
        # Save the label
        labels.append(batch_data[b"labels"][i])

print("Loaded CIFAR-10 training set:")
print(f" - np.shape(images)     {np.shape(images)}")
print(f" - np.shape(labels)     {np.shape(labels)}")
```

::: {.output .stream .stdout}
    Loaded CIFAR-10 training set:
     - np.shape(images)     (0,)
     - np.shape(labels)     (0,)
:::
:::

::: {.cell .markdown id="ZBIVoCQwrZiC"}
Semua gambar sekarang ada di RAM dalam variabel images, dengan meta data
yang sesuai dalam label, dan siap untuk manipulasi. Selanjutnya,
menginstal paket Python yang akan digunakan untuk ketiga metode
tersebut.
:::

::: {.cell .markdown id="074cDuRErlle"}
##Pengaturan untuk Menyimpan Gambar pada Disk Mengatur lingkungan
tersebut untuk metode default menyimpan dan mengakses gambar-gambar ini
dari disk. Artikel ini akan mengasumsikan bahwa telah menginstal Python
3.x di sistem tersebut, dan akan menggunakan Pillow untuk manipulasi
gambar:
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="jtndrkzxqLip" outputId="51ad9ce3-1c0f-471b-aebd-f65c1ac89e14"}
``` python
! pip install Pillow
```

::: {.output .stream .stdout}
    Requirement already satisfied: Pillow in /usr/local/lib/python3.10/dist-packages (9.4.0)
:::
:::

::: {.cell .markdown id="AI6TgMGir37g"}
Melakukan install menggunakan Anaconda:
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="nXyi8DlGskTT" outputId="cef37ea5-eb48-44ef-e007-a9c363d92ed7"}
``` python
! conda install -c conda-forge pillow
```

::: {.output .stream .stdout}
    /bin/bash: line 1: conda: command not found
:::
:::

::: {.cell .markdown id="aThi8SoBwB7I"}
## Getting Started With LMDB
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="CTMfb18RwX4R" outputId="9fb62c2d-e997-45f0-9c80-0ff96a35c529"}
``` python
! pip install lmdb
```

::: {.output .stream .stdout}
    Requirement already satisfied: lmdb in /usr/local/lib/python3.10/dist-packages (1.4.1)
:::
:::

::: {.cell .markdown id="Yl9F70awwxnI"}
Melakukan instal melalui Anaconda:
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="443FHDEJw0UN" outputId="dcca2841-47ee-4a1e-f96f-1e066d01932a"}
``` python
! conda install -c conda-forge python-lmdb
```

::: {.output .stream .stdout}
    /bin/bash: line 1: conda: command not found
:::
:::

::: {.cell .markdown id="I_Qc6FVkxCRW"}
## Getting Started With HDF5
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="eZUor9RGxG05" outputId="4d76bb23-f309-40e1-ab60-54df6016653a"}
``` python
! pip install h5py
```

::: {.output .stream .stdout}
    Requirement already satisfied: h5py in /usr/local/lib/python3.10/dist-packages (3.9.0)
    Requirement already satisfied: numpy>=1.17.3 in /usr/local/lib/python3.10/dist-packages (from h5py) (1.25.2)
:::
:::

::: {.cell .markdown id="oggQHzIExXEt"}
Menginstal secara bergantian melalui Anaconda:
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="-CfvAFVGxY0b" outputId="01870880-7172-414e-deb3-3cdbd053f90c"}
``` python
! conda install -c conda-forge h5py
```

::: {.output .stream .stdout}
    /bin/bash: line 1: conda: command not found
:::
:::

::: {.cell .markdown id="jrvlGrdqyAPT"}
## **Storing a Single Image**
:::

::: {.cell .markdown id="iB6uZbdgySHD"}
Melihat perbandingan kuantitatif dari tugas-tugas dasar yang
dipedulikan: berapa lama waktu yang diperlukan untuk membaca dan menulis
file, dan berapa banyak memori disk yang akan digunakan. Ini juga akan
berfungsi sebagai pengenalan dasar tentang cara kerja metode-metode
tersebut, dengan contoh kode tentang cara menggunakannya.
:::

::: {.cell .code id="5L0jZUQ-yUtk"}
``` python
from pathlib import Path

disk_dir = Path("data/disk/")
lmdb_dir = Path("data/lmdb/")
hdf5_dir = Path("data/hdf5/")
```
:::

::: {.cell .code id="T74YnaWGzDj8"}
``` python
disk_dir.mkdir(parents=True, exist_ok=True)
lmdb_dir.mkdir(parents=True, exist_ok=True)
hdf5_dir.mkdir(parents=True, exist_ok=True)
```
:::

::: {.cell .markdown id="ebMW2skAzTF0"}
## Storing to Disk
:::

::: {.cell .code id="8so8Mw9BzXQP"}
``` python
from PIL import Image
import csv

def store_single_disk(image, image_id, label):
    """ Stores a single image as a .png file on disk.
        Parameters:
        ---------------
        image       image array, (32, 32, 3) to be stored
        image_id    integer unique ID for image
        label       image label
    """
    Image.fromarray(image).save(disk_dir / f"{image_id}.png")

    with open(disk_dir / f"{image_id}.csv", "wt") as csvfile:
        writer = csv.writer(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        writer.writerow([label])
```
:::

::: {.cell .markdown id="lw1bK0oqzeoj"}
## Storing to LMDB
:::

::: {.cell .code id="IKmlHqhCzjkE"}
``` python
class CIFAR_Image:
    def __init__(self, image, label):
        # Dimensions of image for reconstruction - not really necessary
        # for this dataset, but some datasets may include images of
        # varying sizes
        self.channels = image.shape[2]
        self.size = image.shape[:2]

        self.image = image.tobytes()
        self.label = label

    def get_image(self):
        """ Returns the image as a numpy array. """
        image = np.frombuffer(self.image, dtype=np.uint8)
        return image.reshape(*self.size, self.channels)
```
:::

::: {.cell .markdown id="cjSAN4owzqHM"}
Dengan mengingat ketiga hal tersebut, lihatlah kode untuk menyimpan satu
gambar ke LMDB:
:::

::: {.cell .code id="T19ozS-Mz7FT"}
``` python
import lmdb
import pickle

def store_single_lmdb(image, image_id, label):
    """ Stores a single image to a LMDB.
        Parameters:
        ---------------
        image       image array, (32, 32, 3) to be stored
        image_id    integer unique ID for image
        label       image label
    """
    map_size = image.nbytes * 10

    # Create a new LMDB environment
    env = lmdb.open(str(lmdb_dir / f"single_lmdb"), map_size=map_size)

    # Start a new write transaction
    with env.begin(write=True) as txn:
        # All key-value pairs need to be strings
        value = CIFAR_Image(image, label)
        key = f"{image_id:08}"
        txn.put(key.encode("ascii"), pickle.dumps(value))
    env.close()
```
:::

::: {.cell .markdown id="aPMdn5tB0tp-"}
## Storing With HDF5
:::

::: {.cell .code id="IvJw299A0wnb"}
``` python
import h5py

def store_single_hdf5(image, image_id, label):
    """ Stores a single image to an HDF5 file.
        Parameters:
        ---------------
        image       image array, (32, 32, 3) to be stored
        image_id    integer unique ID for image
        label       image label
    """
    # Create a new HDF5 file
    file = h5py.File(hdf5_dir / f"{image_id}.h5", "w")

    # Create a dataset in the file
    dataset = file.create_dataset(
        "image", np.shape(image), h5py.h5t.STD_U8BE, data=image
    )
    meta_set = file.create_dataset(
        "meta", np.shape(label), h5py.h5t.STD_U8BE, data=label
    )
    file.close()
```
:::

::: {.cell .markdown id="b600fc_H1Hwj"}
## Experiment for Storing Many Images
:::

::: {.cell .code id="5O_RWqp11KaQ"}
``` python
_store_single_funcs = dict(
    disk=store_single_disk, lmdb=store_single_lmdb, hdf5=store_single_hdf5
)
```
:::

::: {.cell .markdown id="ic6mDSe21lXe"}
Semuanya sudah siap untuk melakukan percobaan waktu. Lalu mencoba
menyimpan gambar pertama dari CIFAR dan label yang sesuai, dan
menyimpannya dengan tiga cara yang berbeda:
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="FgQqVnyx3ENI" outputId="bc1af80d-88f7-4776-a97f-58f70daa3845"}
``` python
from timeit import timeit

# Example setup for demonstration purposes
images = ["image1", "image2", "image3"]  # Replace with actual image data
labels = ["label1", "label2", "label3"]  # Replace with actual labels

# Example dictionary of functions for demonstration purposes
def store_disk(image, idx, label):
    pass  # Replace with actual storage logic

def store_lmdb(image, idx, label):
    pass  # Replace with actual storage logic

def store_hdf5(image, idx, label):
    pass  # Replace with actual storage logic

_store_single_funcs = {
    "disk": store_disk,
    "lmdb": store_lmdb,
    "hdf5": store_hdf5
}

# Timing dictionary
store_single_timings = dict()

# Timing each method
for method in ("disk", "lmdb", "hdf5"):
    t = timeit(
        "_store_single_funcs[method](image, 0, label)",
        setup="from __main__ import _store_single_funcs, images, labels, method; image=images[0]; label=labels[0]",
        number=1,
    )
    store_single_timings[method] = t
    print(f"Method: {method}, Time usage: {t}")
```

::: {.output .stream .stdout}
    Method: disk, Time usage: 1.4899997040629387e-06
    Method: lmdb, Time usage: 2.2129997887532227e-06
    Method: hdf5, Time usage: 1.99999976757681e-06
:::
:::

::: {.cell .markdown id="zm71jGzF3M-R"}
## **Storing Many Images**
:::

::: {.cell .markdown id="OeKVRdER3VSI"}
## Adjusting the Code for Many Images
:::

::: {.cell .code id="br0m4dK-32Dp"}
``` python
import os
import lmdb
import h5py
import csv
from PIL import Image
import numpy as np
import pickle
from pathlib import Path

# Define directories
disk_dir = Path("/path/to/disk")
lmdb_dir = Path("/path/to/lmdb")
hdf5_dir = Path("/path/to/hdf5")

def store_many_disk(images, labels):
    """ Stores an array of images to disk
        Parameters:
        ---------------
        images       images array, (N, 32, 32, 3) to be stored
        labels       labels array, (N, 1) to be stored
    """
    num_images = len(images)

    # Save all the images one by one
    for i, image in enumerate(images):
        Image.fromarray(image).save(disk_dir / f"{i}.png")

    # Save all the labels to the csv file
    with open(disk_dir / f"{num_images}.csv", "w") as csvfile:
        writer = csv.writer(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        for label in labels:
            # This typically would be more than just one value per row
            writer.writerow([label])

def store_many_lmdb(images, labels):
    """ Stores an array of images to LMDB.
        Parameters:
        ---------------
        images       images array, (N, 32, 32, 3) to be stored
        labels       labels array, (N, 1) to be stored
    """
    num_images = len(images)

    map_size = num_images * images[0].nbytes * 10

    # Create a new LMDB DB for all the images
    env = lmdb.open(str(lmdb_dir / f"{num_images}_lmdb"), map_size=map_size)

    # Same as before — but let's write all the images in a single transaction
    with env.begin(write=True) as txn:
        for i in range(num_images):
            # All key-value pairs need to be Strings
            value = (images[i], labels[i])
            key = f"{i:08}"
            txn.put(key.encode("ascii"), pickle.dumps(value))
    env.close()

def store_many_hdf5(images, labels):
    """ Stores an array of images to HDF5.
        Parameters:
        ---------------
        images       images array, (N, 32, 32, 3) to be stored
        labels       labels array, (N, 1) to be stored
    """
    num_images = len(images)

    # Create a new HDF5 file
    file = h5py.File(hdf5_dir / f"{num_images}_many.h5", "w")

    # Create a dataset in the file
    file.create_dataset(
        "images", np.shape(images), h5py.h5t.STD_U8BE, data=images
    )
    file.create_dataset(
        "meta", np.shape(labels), h5py.h5t.STD_U8BE, data=labels
    )
    file.close()
```
:::

::: {.cell .markdown id="sQqH9ImZ3_nl"}
## Preparing the Dataset
:::

::: {.cell .code colab="{\"base_uri\":\"https://localhost:8080/\"}" id="x7wzsnp14m6W" outputId="a88afe94-e602-4618-d224-edb69e38abaa"}
``` python
cutoffs = [10, 100, 1000, 10000, 100000]

# Let's double our images so that we have 100,000
images = np.concatenate((images, images), axis=0)
labels = np.concatenate((labels, labels), axis=0)

# Make sure you actually have 100,000 images and labels
print(np.shape(images))
print(np.shape(labels))
```

::: {.output .stream .stdout}
    (6,)
    (6,)
:::
:::

::: {.cell .code id="BcN9juLA61bd"}
``` python
_store_many_funcs = dict(
    disk=store_many_disk, lmdb=store_many_lmdb, hdf5=store_many_hdf5
)

from timeit import timeit

store_many_timings = {"disk": [], "lmdb": [], "hdf5": []}

for cutoff in cutoffs:
    for method in ("disk", "lmdb", "hdf5"):
        t = timeit(
            "_store_many_funcs[method](images_, labels_)",
            setup="images_=images[:cutoff]; labels_=labels[:cutoff]",
            number=1,
            globals=globals(),
        )
        store_many_timings[method].append(t)

        # Print out the method, cutoff, and elapsed time
        print(f"Method: {method}, Time usage: {t}")
```
:::

::: {.cell .markdown id="IsULnZeJ6_iM"}
Grafik pertama menunjukkan waktu penyimpanan normal yang tidak
disesuaikan, menyoroti perbedaan drastis antara menyimpan ke file .png
dan LMDB atau HDF5.

Grafik kedua menunjukkan log pengaturan waktu, menyoroti bahwa HDF5
dimulai lebih lambat daripada LMDB tetapi, dengan jumlah gambar yang
lebih besar, sedikit lebih unggul.
:::

::: {.cell .code id="_m5Lwg2g7JBd"}
``` python
import matplotlib.pyplot as plt

def plot_with_legend(
    x_range, y_data, legend_labels, x_label, y_label, title, log=False
):
    """ Displays a single plot with multiple datasets and matching legends.
        Parameters:
        --------------
        x_range         list of lists containing x data
        y_data          list of lists containing y values
        legend_labels   list of string legend labels
        x_label         x axis label
        y_label         y axis label
    """
    plt.style.use("seaborn-whitegrid")
    plt.figure(figsize=(10, 7))

    if len(y_data) != len(legend_labels):
        raise TypeError(
            "Error: number of data sets does not match number of labels."
        )

    all_plots = []
    for data, label in zip(y_data, legend_labels):
        if log:
            temp, = plt.loglog(x_range, data, label=label)
        else:
            temp, = plt.plot(x_range, data, label=label)
        all_plots.append(temp)

    plt.title(title)
    plt.xlabel(x_label)
    plt.ylabel(y_label)
    plt.legend(handles=all_plots)
    plt.show()

# Getting the store timings data to display
disk_x = store_many_timings["disk"]
lmdb_x = store_many_timings["lmdb"]
hdf5_x = store_many_timings["hdf5"]

plot_with_legend(
    cutoffs,
    [disk_x, lmdb_x, hdf5_x],
    ["PNG files", "LMDB", "HDF5"],
    "Number of images",
    "Seconds to store",
    "Storage time",
    log=False,
)

plot_with_legend(
    cutoffs,
    [disk_x, lmdb_x, hdf5_x],
    ["PNG files", "LMDB", "HDF5"],
    "Number of images",
    "Seconds to store",
    "Log storage time",
    log=True,
)
```
:::

::: {.cell .markdown id="rXDF835i7NEZ"}
## **Experiment for Storing Many Images** {#experiment-for-storing-many-images}
:::

::: {.cell .markdown id="YJkEv71F7jC1"}
## Reading From Disk
:::

::: {.cell .code id="gbhCtDmV7opI"}
``` python
def read_single_disk(image_id):
    """ Stores a single image to disk.
        Parameters:
        ---------------
        image_id    integer unique ID for image

        Returns:
        ----------
        image       image array, (32, 32, 3) to be stored
        label       associated meta data, int label
    """
    image = np.array(Image.open(disk_dir / f"{image_id}.png"))

    with open(disk_dir / f"{image_id}.csv", "r") as csvfile:
        reader = csv.reader(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        label = int(next(reader)[0])

    return image, label
```
:::

::: {.cell .markdown id="5aYzzXw-7x35"}
## Reading From LMDB
:::

::: {.cell .code id="QwjjtZts71gV"}
``` python
def read_single_lmdb(image_id):
    """ Stores a single image to LMDB.
        Parameters:
        ---------------
        image_id    integer unique ID for image

        Returns:
        ----------
        image       image array, (32, 32, 3) to be stored
        label       associated meta data, int label
    """
    # Open the LMDB environment
    env = lmdb.open(str(lmdb_dir / f"single_lmdb"), readonly=True)

    # Start a new read transaction
    with env.begin() as txn:
        # Encode the key the same way as we stored it
        data = txn.get(f"{image_id:08}".encode("ascii"))
        # Remember it's a CIFAR_Image object that is loaded
        cifar_image = pickle.loads(data)
        # Retrieve the relevant bits
        image = cifar_image.get_image()
        label = cifar_image.label
    env.close()

    return image, label
```
:::

::: {.cell .markdown id="YpD9zWpk75gh"}
## Reading From HDF5
:::

::: {.cell .code id="KeDJp7dq79Sm"}
``` python
def read_single_hdf5(image_id):
    """ Stores a single image to HDF5.
        Parameters:
        ---------------
        image_id    integer unique ID for image

        Returns:
        ----------
        image       image array, (32, 32, 3) to be stored
        label       associated meta data, int label
    """
    # Open the HDF5 file
    file = h5py.File(hdf5_dir / f"{image_id}.h5", "r+")

    image = np.array(file["/image"]).astype("uint8")
    label = int(np.array(file["/meta"]).astype("uint8"))

    return image, label
```
:::

::: {.cell .code id="jdw8dEx98BoN"}
``` python
_read_single_funcs = dict(
    disk=read_single_disk, lmdb=read_single_lmdb, hdf5=read_single_hdf5
)
```
:::

::: {.cell .markdown id="iNqDumgb8IEK"}
## Experiment for Reading a Single Image
:::

::: {.cell .code id="pC6rbs7G8ST4"}
``` python
from timeit import timeit

read_single_timings = dict()

for method in ("disk", "lmdb", "hdf5"):
    t = timeit(
        "_read_single_funcs[method](0)",
        setup="image=images[0]; label=labels[0]",
        number=1,
        globals=globals(),
    )
    read_single_timings[method] = t
    print(f"Method: {method}, Time usage: {t}")
```
:::

::: {.cell .markdown id="axyZwCKT8gh0"}
## **Reading Many Images**
:::

::: {.cell .markdown id="PT5LrkWO8mXQ"}
## Adjusting the Code for Many Images {#adjusting-the-code-for-many-images}
:::

::: {.cell .code id="c7Mg5iSq8sU8"}
``` python
def read_many_disk(num_images):
    """ Reads image from disk.
        Parameters:
        ---------------
        num_images   number of images to read

        Returns:
        ----------
        images      images array, (N, 32, 32, 3) to be stored
        labels      associated meta data, int label (N, 1)
    """
    images, labels = [], []

    # Loop over all IDs and read each image in one by one
    for image_id in range(num_images):
        images.append(np.array(Image.open(disk_dir / f"{image_id}.png")))

    with open(disk_dir / f"{num_images}.csv", "r") as csvfile:
        reader = csv.reader(
            csvfile, delimiter=" ", quotechar="|", quoting=csv.QUOTE_MINIMAL
        )
        for row in reader:
            labels.append(int(row[0]))
    return images, labels

def read_many_lmdb(num_images):
    """ Reads image from LMDB.
        Parameters:
        ---------------
        num_images   number of images to read

        Returns:
        ----------
        images      images array, (N, 32, 32, 3) to be stored
        labels      associated meta data, int label (N, 1)
    """
    images, labels = [], []
    env = lmdb.open(str(lmdb_dir / f"{num_images}_lmdb"), readonly=True)

    # Start a new read transaction
    with env.begin() as txn:
        # Read all images in one single transaction, with one lock
        # We could split this up into multiple transactions if needed
        for image_id in range(num_images):
            data = txn.get(f"{image_id:08}".encode("ascii"))
            # Remember that it's a CIFAR_Image object
            # that is stored as the value
            cifar_image = pickle.loads(data)
            # Retrieve the relevant bits
            images.append(cifar_image.get_image())
            labels.append(cifar_image.label)
    env.close()
    return images, labels

def read_many_hdf5(num_images):
    """ Reads image from HDF5.
        Parameters:
        ---------------
        num_images   number of images to read

        Returns:
        ----------
        images      images array, (N, 32, 32, 3) to be stored
        labels      associated meta data, int label (N, 1)
    """
    images, labels = [], []

    # Open the HDF5 file
    file = h5py.File(hdf5_dir / f"{num_images}_many.h5", "r+")

    images = np.array(file["/images"]).astype("uint8")
    labels = np.array(file["/meta"]).astype("uint8")

    return images, labels

_read_many_funcs = dict(
    disk=read_many_disk, lmdb=read_many_lmdb, hdf5=read_many_hdf5
)
```
:::

::: {.cell .markdown id="6dltMzPc8t_4"}
## Experiment for Reading Many Images
:::

::: {.cell .code id="BHDRD9lK8wse"}
``` python
from timeit import timeit

read_many_timings = {"disk": [], "lmdb": [], "hdf5": []}

for cutoff in cutoffs:
    for method in ("disk", "lmdb", "hdf5"):
        t = timeit(
            "_read_many_funcs[method](num_images)",
            setup="num_images=cutoff",
            number=1,
            globals=globals(),
        )
        read_many_timings[method].append(t)

        # Print out the method, cutoff, and elapsed time
        print(f"Method: {method}, No. images: {cutoff}, Time usage: {t}")
```
:::

::: {.cell .markdown id="7PPgVhOg-Lqd"}
Ada beberapa fitur pembeda lain dari LMDB dan HDF5 yang perlu diketahui,
dan juga penting untuk membahas secara singkat beberapa kritik terhadap
kedua metode tersebut. Beberapa tautan disertakan bersama dengan diskusi
jika Anda ingin mempelajari lebih lanjut.

-   Akses Paralel
    Perbandingan utama yang tidak diuji dalam percobaan di atas adalah
    pembacaan dan penulisan secara bersamaan. Sering kali, dengan
    kumpulan data yang besar, mungkin ingin mempercepat operasi melalui
    paralelisasi.

-   Dokumentasi
    Jika Google lmdb, setidaknya di Inggris, hasil pencarian ketiga
    adalah IMDb, Internet Movie Database.

-   Pandangan yang Lebih Kritis pada Implementasi
    Poin penting yang harus dipahami tentang LMDB adalah bahwa data baru
    ditulis tanpa menimpa atau memindahkan data yang sudah ada. Ini
    adalah keputusan desain yang memungkinkan pembacaan yang sangat
    cepat seperti yang Anda saksikan dalam percobaan kami, dan juga
    menjamin integritas dan keandalan data tanpa perlu menyimpan catatan
    transaksi.
:::

::: {.cell .code id="IGOshDqN-ZOY"}
``` python
# Slightly slower
for i in range(len(dataset)):
    # Read the ith value in the dataset, one at a time
    do_something_with(dataset[i])

# This is better
data = dataset[:]
for d in data:
    do_something_with(d)
```
:::