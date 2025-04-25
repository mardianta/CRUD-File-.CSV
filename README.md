# CRUD-File-.CSV

```
  <?php

  // Konfigurasi file CSV
  $csvFile = 'data.csv';
  $delimiter = ',';
  $enclosure = '"';
  
  // Fungsi untuk membaca file CSV dan mengembalikannya sebagai array
  function readCSV($filename, $delimiter = ',', $enclosure = '"', $escape = "\\") {
      $data = [];
      if (($handle = fopen($filename, 'r')) !== false) {
          while (($row = fgetcsv($handle, 0, $delimiter, $enclosure, $escape)) !== false) {
              $data[] = $row;
          }
          fclose($handle);
      }
      return $data;
  }
  
  // Fungsi untuk menulis array data ke file CSV
  function writeCSV($filename, $data, $delimiter = ',', $enclosure = '"', $escape = "\\") {
      if (($handle = fopen($filename, 'w')) !== false) {
          foreach ($data as $row) {
              if (is_array($row)) {
                  fputcsv($handle, $row, $delimiter, $enclosure, $escape);
              }
          }
          fclose($handle);
          return true;
      }
      return false;
  }
  
  // Baca data CSV
  $csvData = readCSV($csvFile, $delimiter, $enclosure);
  
  // Proses penghapusan kolom
  if (isset($_POST['delete_column'])) {
      $columnIndexToDelete = $_POST['delete_column_index'];
      if (isset($csvData[0][$columnIndexToDelete])) {
          array_splice($csvData[0], $columnIndexToDelete, 1);
          for ($i = 1; $i < count($csvData); $i++) {
              if (isset($csvData[$i][$columnIndexToDelete])) {
                  array_splice($csvData[$i], $columnIndexToDelete, 1);
              }
          }
          if (writeCSV($csvFile, $csvData, $delimiter, $enclosure)) {
              echo '<div style="color: green;">Kolom berhasil dihapus. <a href="">Muat ulang halaman</a></div>';
          } else {
              echo '<div style="color: red;">Gagal menghapus kolom.</div>';
          }
      } else {
          echo '<div style="color: red;">Indeks kolom tidak valid.</div>';
      }
  }
  
  // Proses penambahan kolom (dari tombol plus di header)
  if (isset($_POST['add_new_column'])) {
      if (!empty($csvData)) {
          $csvData[0][] = 'null'; // Tambahkan 'null' di akhir header
          for ($i = 1; $i < count($csvData); $i++) {
              $csvData[$i][] = 'null'; // Tambahkan 'null' di akhir setiap baris data
          }
      } else {
          $csvData[0] = ['null']; // Jika file kosong, buat header baru dengan 'null'
      }
  
      if (writeCSV($csvFile, $csvData, $delimiter, $enclosure)) {
          echo '<div style="color: green;">Kolom baru berhasil ditambahkan. <a href="">Muat ulang halaman</a></div>';
      } else {
          echo '<div style="color: red;">Gagal menambahkan kolom baru.</div>';
      }
  }
  
  // Proses pembaruan data secara langsung
  if (isset($_POST['update_cell'])) {
      $rowIndex = $_POST['row_index'];
      $columnIndex = $_POST['column_index'];
      $newValue = isset($_POST['new_value']) ? trim($_POST['new_value']) : ''; // Trim nilai yang diterima
      if (isset($csvData[$rowIndex][$columnIndex])) {
          $csvData[$rowIndex][$columnIndex] = $newValue;
          writeCSV($csvFile, $csvData, $delimiter, $enclosure);
      }
      exit;
  }
  
  // Proses penambahan baris (dari tombol plus di footer)
  if (isset($_POST['add_new_row'])) {
      if (!empty($csvData[0])) {
          $newRow = array_fill(0, count($csvData[0]), 'null');
          $csvData[] = $newRow;
          if (writeCSV($csvFile, $csvData, $delimiter, $enclosure)) {
              echo '<div style="color: green;">Baris baru berhasil ditambahkan. <a href="">Muat ulang halaman</a></div>';
          } else {
              echo '<div style="color: red;">Gagal menambahkan baris baru.</div>';
          }
      } else {
          echo '<div style="color: orange;">Tidak dapat menambahkan baris karena tidak ada kolom. Tambahkan kolom terlebih dahulu.</div>';
      }
  }
  
  // Proses penghapusan baris
  if (isset($_POST['delete_row'])) {
      $deleteIndex = $_POST['delete_index'];
      if (isset($csvData[$deleteIndex])) {
          unset($csvData[$deleteIndex]);
          $csvData = array_values($csvData);
          if (writeCSV($csvFile, $csvData, $delimiter, $enclosure)) {
              echo '<div style="color: green;">Baris berhasil dihapus. <a href="">Muat ulang halaman</a></div>';
          } else {
              echo '<div style="color: red;">Gagal menghapus baris.</div>';
          }
      } else {
          echo '<div style="color: red;">Indeks baris tidak valid.</div>';
      }
  }
  
  ?>
  
  <!DOCTYPE html>
  <html lang="id">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Pengelola Data CSV</title>
      <style>
          #csvTable {
              border-collapse: collapse;
              width: 100%;
          }
  
          #csvTable th, #csvTable td {
              border: 1px solid #ddd;
              padding: 8px;
              white-space: nowrap;
              position: relative; /* Untuk ikon hapus di nomor baris */
              text-align: center;
          }
  
          #csvTable th:first-child {
              padding-left: 25px; /* Ruang untuk ikon hapus baris */
          }
  
          .resizer {
              position: absolute;
              top: 0;
              right: 15px;
              bottom: 0;
              width: 10px;
              background: #007bff;
              cursor: col-resize;
              user-select: none;
          }
  
          .resizer.active {
              background: #0056b3;
          }
  
          .delete-column-btn, .delete-row-btn, .add-column-btn, .add-row-btn {
              background: none;
              border: none;
              color: red;
              cursor: pointer;
              font-size: 16px;
              padding: 0;
              margin: 0;
              outline: none;
              line-height: 1;
          }
  
          .delete-column-btn {
              position: absolute;
              top: 5px;
              right: 2px;
          }
  
          .add-column-btn {
              position: absolute;
              top: 5px;
              right: 2px;
              color: green; /* Warna ikon tambah kolom */
              font-size: 14px;
          }
  
          .delete-row-btn {
              position: absolute;
              top: 5px;
              left: 5px;
          }
  
          .add-row-btn {
              display: block;
              width: 100%;
              padding: 8px;
              text-align: center;
              background-color: #f2f2f2;
              border-top: 1px solid #ddd;
              cursor: pointer;
              color: green;
              font-size: 14px;
          }
  
          .add-row-btn:hover {
              background-color: #e0e0e0;
          }
  
          #add-column-container {
              position: relative;
              text-align: right;
          }
      </style>
  </head>
  <body>
  
      <h2>Pengelola Data CSV</h2>
  
      <table border="1" id="csvTable">
          <thead>
              <tr>
                  <th></th>
                  <?php if (!empty($csvData[0])): ?>
                      <?php foreach ($csvData[0] as $columnIndex => $columnName): ?>
                          <th>
                              <?php echo htmlspecialchars(chr(65 + $columnIndex)); ?>
                              <form method="post" style="display:inline;">
                                  <input type="hidden" name="delete_column" value="1">
                                  <input type="hidden" name="delete_column_index" value="<?php echo $columnIndex; ?>">
                                  <button type="submit" class="delete-column-btn" onclick="return confirm('Apakah Anda yakin ingin menghapus kolom ini?');">x</button>
                                  <div class="resizer" data-col-index="<?php echo $columnIndex; ?>"></div>
                              </form>
                          </th>
                      <?php endforeach; ?>
                  <?php endif; ?>
                  <th>
                      <div id="add-column-container">
                          <form method="post" style="display:inline;">
                              <input type="hidden" name="add_new_column" value="1">
                              <button type="submit" class="add-column-btn">+</button>
                          </form>
                      </div>
                  </th>
              </tr>
          </thead>
          <tbody>
              <?php if (!empty($csvData)): ?>
                  <?php foreach ($csvData as $rowIndex => $rowData): ?>
                      <tr data-row-index="<?php echo $rowIndex; ?>">
                          <th>
                              <?php echo ($rowIndex + 1); ?>
                              <form method="post" style="display:inline;">
                                  <input type="hidden" name="delete_row" value="1">
                                  <input type="hidden" name="delete_index" value="<?php echo $rowIndex; ?>">
                                  <button type="submit" class="delete-row-btn" onclick="return confirm('Apakah Anda yakin ingin menghapus baris ini?');">x</button>
                              </form>
                          </th>
                          <?php if (!empty($csvData[0])): ?>
                              <?php foreach ($rowData as $columnIndex => $cell): ?>
                                  <td contenteditable="true"
                                      data-row="<?php echo $rowIndex; ?>"
                                      data-col="<?php echo $columnIndex; ?>">
                                      <?php echo htmlspecialchars($cell); ?>
                                  </td>
                              <?php endforeach; ?>
                          <?php endif; ?>
                      </tr>
                  <?php endforeach; ?>
              <?php else: ?>
                  <tr><td>File CSV kosong atau tidak ditemukan.</td></tr>
              <?php endif; ?>
          </tbody>
          <tfoot>
              <tr>
                  <td colspan="<?php echo (empty($csvData[0]) ? 1 : count($csvData[0]) + 1); ?>">
                      <form method="post">
                          <input type="hidden" name="add_new_row" value="1">
                          <button type="submit" class="add-row-btn">+</button>
                      </form>
                  </td>
              </tr>
          </tfoot>
      </table>
  
      <script>
          document.addEventListener("DOMContentLoaded", function() {
              const csvTable = document.getElementById("csvTable");
              const bodyCells = csvTable.querySelectorAll('tbody td[contenteditable="true"]');
              const headers = csvTable.getElementsByTagName("th");
              let currentResizer = null;
              let startX;
              let startWidth;
  
              // Resizer kolom (hanya pada th yang memiliki resizer)
              for (let i = 0; i < headers.length - 1; i++) {
                  const resizer = headers[i].querySelector(".resizer");
                  if (resizer) {
                      resizer.addEventListener("mousedown", function(e) {
                          currentResizer = e.target;
                          startX = e.clientX;
                          startWidth = headers[i].offsetWidth;
                          currentResizer.classList.add("active");
                          document.addEventListener("mousemove", resizeColumn);
                          document.addEventListener("mouseup", stopResize);
                      });
                  }
              }
  
              function resizeColumn(e) {
                  if (currentResizer) {
                      const newWidth = startWidth + (e.clientX - startX);
                      const columnIndex = parseInt(currentResizer.getAttribute("data-col-index"));
                      const headerCell = currentResizer.parentNode.parentNode;
                      headerCell.style.width = newWidth + "px";
                      const rows = csvTable.getElementsByTagName("tr");
                      for (let i = 0; i < rows.length; i++) {
                          const cells = rows[i].getElementsByTagName("td");
                          if (cells.length > columnIndex) {
                              cells[columnIndex].style.width = newWidth + "px";
                          }
                      }
                  }
              }
  
              function stopResize() {
                  if (currentResizer) {
                      currentResizer.classList.remove("active");
                      document.removeEventListener("mousemove", resizeColumn);
                      document.removeEventListener("mouseup", stopResize);
                      currentResizer = null;
                  }
              }
  
              // Edit langsung di sel (hanya pada td yang contenteditable)
              bodyCells.forEach(cell => {
                  cell.addEventListener('blur', function() {
                      const rowIndex = this.getAttribute('data-row');
                      const columnIndex = this.getAttribute('data-col');
                      const newValue = this.textContent;
  
                      fetch('', {
                          method: 'POST',
                          headers: {
                              'Content-Type': 'application/x-www-form-urlencoded',
                          },
                          body: `update_cell=1&row_index=${rowIndex}&column_index=${columnIndex}&new_value=${encodeURIComponent(newValue)}`,
                      });
                  });
              });
          });
      </script>
  
  </body>
  </html>
```
