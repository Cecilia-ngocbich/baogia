
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.5">
    <title>Phần Mềm Báo Giá Chuyên Nghiệp</title>
    <script src="https://cdn.sheetjs.com/xlsx-0.20.0/package/dist/xlsx.full.min.js"></script>
    
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; padding: 20px; background-color: #f5f5f5; }
        h2 { color: #2c3e50; }
        
        /* Khu vực nhập liệu chung */
        .control-panel { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-bottom: 20px; }
        .file-upload { margin-bottom: 15px; padding-bottom: 15px; border-bottom: 1px dashed #ccc; }
        
        .input-row { display: flex; flex-wrap: wrap; gap: 10px; align-items: center; margin-bottom: 10px; }
        input, select { padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
        input[readonly] { background-color: #e9ecef; }
        
        button { padding: 10px 20px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; transition: 0.3s; }
        .btn-add { background-color: #28a745; color: white; }
        .btn-add:hover { background-color: #218838; }
        .btn-action { background-color: #007bff; color: white; margin-right: 10px; }
        .btn-action:hover { background-color: #0069d9; }

        /* Bảng hiển thị */
        table { width: 100%; border-collapse: collapse; background: white; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        th, td { border: 1px solid #dee2e6; padding: 12px; text-align: left; }
        th { background-color: #343a40; color: white; }
        .text-right { text-align: right; }
        .text-center { text-align: center; }
        .del-btn { color: #dc3545; cursor: pointer; font-weight: bold; text-align: center; }
        
        /* Khu vực thông tin báo giá & tổng tiền */
        .info-header { display: flex; justify-content: space-between; margin-bottom: 15px; }
        .customer-info { flex: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; margin-right: 20px;}
        .quote-meta { width: 250px; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
        .quote-meta p { margin: 5px 0; font-size: 1.1em; }
        
        .summary-section { margin-top: 20px; float: right; width: 400px; }
        .summary-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #eee; }
        .summary-row.final { font-weight: bold; font-size: 1.2em; border-top: 2px solid #333; color: #d63384; }

        /* CSS dành cho in ấn */
        @media print {
            .control-panel, .btn-action, .del-btn, .file-upload, #export-pdf-btn, #export-excel-btn {
                display: none !important; /* Ẩn các nút và khu vực điều khiển */
            }
            body { background-color: white !important; padding: 0 !important; }
            table, th, td { border-color: #000 !important; } /* Đảm bảo đường viền đen */
            .summary-section { float: none; width: 100%; } /* Đưa tổng tiền về full width */
            .quote-meta { border: none; }
        }

    </style>
</head>
<body>

    <h2>PHẦN MỀM BÁO GIÁ CÔNG TY TÙNG LÂM</h2>

    <div class="info-header">
        <div class="customer-info">
            <div class="input-row">
                <label style="width: 120px;">Tên Khách Hàng:</label>
                <input type="text" id="cust-name" placeholder="Công ty ABC..." style="flex: 1;">
                <label style="width: 50px; margin-left: 20px;">MST:</label>
                <input type="text" id="cust-mst" placeholder="0123456789" style="flex: 1;">
            </div>
            <div class="input-row">
                <label style="width: 120px;">Người liên hệ:</label>
                <input type="text" id="cust-contact" placeholder="Ông/Bà..." style="flex: 1;">
                <label style="width: 50px; margin-left: 20px;">SĐT:</label>
                <input type="text" id="cust-phone" placeholder="0901xxxxxx" style="flex: 1;">
            </div>
        </div>
        <div class="quote-meta">
            <p><strong>Số Báo Giá:</strong> <span id="quote-number"></span></p>
            <p><strong>Ngày Báo Giá:</strong> <span id="quote-date"></span></p>
        </div>
    </div>
    
    <div class="control-panel">
        <div class="file-upload">
            <label><strong>1. Nhập dữ liệu sản phẩm (Excel):</strong> </label>
            <input type="file" id="upload-file" accept=".xlsx, .xls" onchange="importExcel(this)">
            <small style="color:gray">File cần có cột: Code, Name, Unit, Price, Brand, VAT</small>
        </div>

        <div class="input-row">
            <div style="flex: 2;">
                <input list="product-list" id="inp-product" placeholder="Gõ tên hàng để tìm..." style="width: 95%;" onchange="autoFillInfo()">
                <datalist id="product-list"></datalist>
            </div>
            <input type="text" id="inp-code" placeholder="Mã hàng" readonly style="width: 80px;">
            <input type="text" id="inp-unit" placeholder="Đơn vị" readonly style="width: 60px;">
            <input type="text" id="inp-vat" placeholder="VAT%" readonly style="width: 50px;">
            <input type="text" id="inp-price" placeholder="Đơn giá" readonly style="width: 100px;">
            <input type="number" id="inp-qty" placeholder="Nhập SL" value="1" style="width: 60px;">
            <input type="number" id="inp-discount" placeholder="Nhập % CK" value="0" style="width: 60px;">
            <input type="text" id="inp-brand" placeholder="Thương hiệu" readonly style="width: 100px;">
            <button class="btn-add" onclick="addToQuote()">+ Thêm</button>
        </div>
    </div>

    <table id="quote-table">
        <thead>
            <tr>
                <th>Mã Hàng</th>
                <th>Tên Hàng Hóa</th>
                <th>Đơn vị</th>
                <th>SL</th>
                <th>Đơn Giá Gốc</th>
                <th>% CK</th>
                <th>Đơn Giá Sau CK</th>
                <th>VAT (%)</th>
                <th>Thành Tiền (Trước VAT)</th>
                <th>Thương Hiệu</th>
                <th class="del-btn">Xóa</th>
            </tr>
        </thead>
        <tbody id="quote-body">
            </tbody>
    </table>

    <div class="summary-section">
        <div class="summary-row">
            <span>Tổng tiền hàng:</span>
            <span id="txt-subtotal">0</span>
        </div>
        <div class="summary-row">
            <span>VAT (8%):</span>
            <span id="txt-vat8">0</span>
        </div>
        <div class="summary-row">
            <span>VAT (10%):</span>
            <span id="txt-vat10">0</span>
        </div>
        <div class="summary-row final">
            <span>TỔNG THANH TOÁN:</span>
            <span id="txt-final">0</span>
        </div>
    </div>

    <div style="clear: both;"></div>
    <div style="margin-top: 20px;">
        <button id="export-excel-btn" class="btn-action" onclick="exportToExcel()">📥 Tải Bảng Báo Giá (.xlsx)</button>
        <button id="export-pdf-btn" class="btn-action" onclick="window.print()">🖨️ In ra PDF/Giấy</button>
    </div>

    <script>
    // Biến lưu database sản phẩm từ Excel (sẽ được lưu vào LocalStorage)
    let productDatabase = [];
    // Biến lưu danh sách các mặt hàng đang báo giá để tính toán
    let quoteItems = [];
    // Key dùng để lưu trữ dữ liệu trong LocalStorage
    const DB_STORAGE_KEY = 'productDatabase_latest';
    const QUOTE_NUMBER_KEY = 'quoteNumber_current'; // Dùng để lưu trữ số báo giá

    // Chạy khi trang load
    document.addEventListener('DOMContentLoaded', (event) => {
        // 1. Tải dữ liệu danh mục sản phẩm lần cuối cùng
        loadProductDatabaseFromStorage();
        // 2. Thiết lập Số báo giá và Ngày báo giá
        setQuoteMetadata();
    });

    // ---------------------------------------------------------
    // CHỨC NĂNG LƯU/TẢI DỮ LIỆU DANH MỤC TỪ LOCALSTORAGE
    // ---------------------------------------------------------

    function saveProductDatabaseToStorage() {
        if (productDatabase.length > 0) {
            try {
                // Lưu dữ liệu dưới dạng chuỗi JSON
                localStorage.setItem(DB_STORAGE_KEY, JSON.stringify(productDatabase));
                console.log("Database đã được lưu vào LocalStorage.");
            } catch (e) {
                console.error("Không thể lưu LocalStorage:", e);
            }
        }
    }

    function loadProductDatabaseFromStorage() {
        const savedData = localStorage.getItem(DB_STORAGE_KEY);
        if (savedData) {
            try {
                productDatabase = JSON.parse(savedData);
                // Cập nhật Datalist sau khi tải
                updateDatalist();
                alert(`Đã tải thành công ${productDatabase.length} sản phẩm từ dữ liệu đã lưu gần nhất!`);
            } catch (e) {
                console.error("Lỗi khi tải dữ liệu từ LocalStorage:", e);
            }
        } else {
            console.log("Không tìm thấy dữ liệu sản phẩm đã lưu.");
        }
    }

    function updateDatalist() {
        const dataList = document.getElementById('product-list');
        dataList.innerHTML = '';
        productDatabase.forEach(p => {
            let option = document.createElement('option');
            option.value = p.name;
            dataList.appendChild(option);
        });
    }

    // ---------------------------------------------------------
    // CHỨC NĂNG SỐ BÁO GIÁ
    // ---------------------------------------------------------
    function setQuoteMetadata() {
        const now = new Date();
        
        // Ngày báo giá (DD/MM/YYYY)
        const day = String(now.getDate()).padStart(2, '0');
        const month = String(now.getMonth() + 1).padStart(2, '0');
        const year = now.getFullYear();
        document.getElementById('quote-date').innerText = `${day}/${month}/${year}`;

        // Số báo giá (MMYY###) - Lấy từ LocalStorage hoặc khởi tạo
        const month_str = String(now.getMonth() + 1).padStart(2, '0');
        const year_short = String(now.getFullYear()).slice(-2);
        const prefix = `${month_str}${year_short}`;

        let currentQuoteNum = localStorage.getItem(QUOTE_NUMBER_KEY);
        
        if (!currentQuoteNum || !currentQuoteNum.startsWith(prefix)) {
            // Nếu là tháng/năm mới, hoặc chưa có số, reset về 001
            currentQuoteNum = `${prefix}001`;
        } else {
            // Tăng số thứ tự nếu là tháng/năm cũ
            const serial = parseInt(currentQuoteNum.slice(-3));
            currentQuoteNum = `${prefix}${(serial).toString().padStart(3, '0')}`;
        }
        
        document.getElementById('quote-number').innerText = currentQuoteNum;
        // Lưu số báo giá hiện tại, sẽ tăng khi xuất file
        localStorage.setItem(QUOTE_NUMBER_KEY, currentQuoteNum); 
    }


    // ---------------------------------------------------------
    // 3. CHỨC NĂNG IMPORT EXCEL (Đã điều chỉnh)
    // ---------------------------------------------------------
    function importExcel(input) {
        const file = input.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            const data = new Uint8Array(e.target.result);
            const workbook = XLSX.read(data, {type: 'array'});
            
            const firstSheetName = workbook.SheetNames[0];
            const worksheet = workbook.Sheets[firstSheetName];
            
            const jsonData = XLSX.utils.sheet_to_json(worksheet);
            
            productDatabase = jsonData.map(item => ({
                code: item.Code || item.code || "",
                name: item.Name || item.name || "",
                unit: item.Unit || item.unit || "",
                price: item.Price || item.price || 0,
                brand: item.Brand || item.brand || "",
                vat: item.VAT || item.vat || 0
            }));

            updateDatalist(); // Cập nhật Datalist
            saveProductDatabaseToStorage(); // *** LƯU DỮ LIỆU MỚI VÀO LOCALSTORAGE ***

            alert("Đã nhập thành công " + productDatabase.length + " sản phẩm và lưu vào bộ nhớ trình duyệt!");
        };
        reader.readAsArrayBuffer(file);
    }

    // ---------------------------------------------------------
    // CÁC HÀM CÒN LẠI (GIỮ NGUYÊN)
    // ---------------------------------------------------------

    function autoFillInfo() {
        let val = document.getElementById('inp-product').value;
        let product = productDatabase.find(p => p.name === val);
        
        if (product) {
            document.getElementById('inp-code').value = product.code;
            document.getElementById('inp-unit').value = product.unit;
            document.getElementById('inp-price').value = product.price.toLocaleString(); 
            document.getElementById('inp-brand').value = product.brand;
            document.getElementById('inp-vat').value = product.vat;
        }
    }

    function addToQuote() {
        let nameStr = document.getElementById('inp-product').value;
        let productRaw = productDatabase.find(p => p.name === nameStr);
        
        if (!productRaw) {
            alert("Vui lòng chọn sản phẩm đúng từ danh sách!");
            return;
        }

        let qty = parseFloat(document.getElementById('inp-qty').value) || 0;
        let discountPercent = parseFloat(document.getElementById('inp-discount').value) || 0;
        let priceRaw = productRaw.price;
        let vatRate = parseFloat(productRaw.vat);

        let priceAfterDisc = Math.round(priceRaw * (1 - discountPercent/100));
        let lineTotal = Math.round(priceAfterDisc * qty);
        
        let vatAmount = 0;
        if (vatRate === 8) vatAmount = Math.round(lineTotal * 0.08);
        if (vatRate === 10) vatAmount = Math.round(lineTotal * 0.10);

        let item = {
            id: Date.now(),
            code: productRaw.code,
            name: productRaw.name,
            unit: productRaw.unit,
            brand: productRaw.brand,
            priceRaw: priceRaw,
            qty: qty,
            discount: discountPercent,
            priceAfterDisc: priceAfterDisc, 
            vatRate: vatRate,
            lineTotal: lineTotal, 
            vatAmount: vatAmount
        };

        quoteItems.push(item);
        renderTable();
        
        document.getElementById('inp-product').value = '';
        document.getElementById('inp-code').value = '';
        document.getElementById('inp-unit').value = '';
        document.getElementById('inp-price').value = '';
        document.getElementById('inp-brand').value = '';
        document.getElementById('inp-vat').value = '';
    }

    function removeRow(id) {
        quoteItems = quoteItems.filter(i => i.id !== id);
        renderTable();
    }

    function renderTable() {
        let tbody = document.getElementById('quote-body');
        tbody.innerHTML = '';

        let totalSub = 0;
        let totalVat8 = 0;
        let totalVat10 = 0;

        quoteItems.forEach(item => {
            totalSub += item.lineTotal;
            if(item.vatRate == 8) totalVat8 += item.vatAmount;
            if(item.vatRate == 10) totalVat10 += item.vatAmount;

            let row = tbody.insertRow();
            row.innerHTML = `
                <td>${item.code}</td>
                <td>${item.name}</td>
                <td class="text-center">${item.unit}</td>
                <td class="text-center">${item.qty}</td>
                <td class="text-right">${item.priceRaw.toLocaleString()}</td>
                <td class="text-center">${item.discount}%</td>
                <td class="text-right">${item.priceAfterDisc.toLocaleString()}</td>
                <td class="text-right">${item.vatRate}%</td>
                <td class="text-right">${item.lineTotal.toLocaleString()}</td>
                <td class="text-center">${item.brand}</td>
                <td class="del-btn" onclick="removeRow(${item.id})">Xóa</td>
            `;
        });

        let totalFinal = totalSub + totalVat8 + totalVat10;
        
        document.getElementById('txt-subtotal').innerText = totalSub.toLocaleString();
        document.getElementById('txt-vat8').innerText = totalVat8.toLocaleString();
        document.getElementById('txt-vat10').innerText = totalVat10.toLocaleString();
        document.getElementById('txt-final').innerText = totalFinal.toLocaleString() + " VNĐ";
    }

    function exportToExcel() {
        if (quoteItems.length === 0) {
            alert("Chưa có dữ liệu để xuất!");
            return;
        }

        // Tăng số báo giá lên 1 và lưu vào LocalStorage sau khi xuất thành công
        incrementQuoteNumber();

        let dataExport = [
            ["Mã Hàng", "Tên Hàng Hóa", "Đơn vị", "Số Lượng", "Đơn Giá", "% CK", "Đơn Giá Sau CK", "VAT%", "Thành Tiền", "Thương Hiệu"]
        ];

        quoteItems.forEach(i => {
            dataExport.push([
                i.code, i.name, i.unit, i.qty, i.priceRaw, i.discount/100, i.priceAfterDisc, i.vatRate/100, i.lineTotal, i.brand
            ]);
        });

        const custName = document.getElementById('cust-name').value || 'Khách Hàng';
        const quoteNum = document.getElementById('quote-number').innerText;

        let header = [
            [`BÁO GIÁ SỐ: ${quoteNum}`],
            [`Khách Hàng: ${custName}`],
            ["", "", "", "", "", "", "", "", "", ""],
        ];
        
        dataExport = header.concat(dataExport);


        let subTotal = parseFloat(document.getElementById('txt-subtotal').innerText.replace(/\./g,'').replace(/,/g,''));
        let vat8 = parseFloat(document.getElementById('txt-vat8').innerText.replace(/\./g,'').replace(/,/g,''));
        let vat10 = parseFloat(document.getElementById('txt-vat10').innerText.replace(/\./g,'').replace(/,/g,''));
        let finalTotal = parseFloat(document.getElementById('txt-final').innerText.replace(/\D/g,''));

        dataExport.push(["", "", "", "", "", "", "", "", "", ""]);
        dataExport.push(["", "", "", "", "", "", "", "Tổng tiền hàng:", subTotal, ""]);
        dataExport.push(["", "", "", "", "", "", "", "VAT 8%:", vat8, ""]);
        dataExport.push(["", "", "", "", "", "", "", "VAT 10%:", vat10, ""]);
        dataExport.push(["", "", "", "", "", "", "", "TỔNG THANH TOÁN:", finalTotal, ""]);

        let wb = XLSX.utils.book_new();
        let ws = XLSX.utils.aoa_to_sheet(dataExport);

        XLSX.utils.book_append_sheet(wb, ws, "Bao Gia");

        XLSX.writeFile(wb, `Bao_Gia_${quoteNum}.xlsx`);
    }

    // Hàm Tăng số báo giá sau khi xuất file
    function incrementQuoteNumber() {
        const currentQuoteNum = document.getElementById('quote-number').innerText;
        const prefix = currentQuoteNum.slice(0, 4); // MMYY
        const serial = parseInt(currentQuoteNum.slice(-3));
        
        // Tăng số thứ tự
        const nextSerial = (serial + 1).toString().padStart(3, '0');
        const nextQuoteNum = `${prefix}${nextSerial}`;
        
        // Lưu số mới vào LocalStorage
        localStorage.setItem(QUOTE_NUMBER_KEY, nextQuoteNum);
        
        // Cập nhật hiển thị số báo giá để chuẩn bị cho lần báo giá tiếp theo
        document.getElementById('quote-number').innerText = nextQuoteNum; 
    }

    function printQuote() {
        window.print();
    }
</script>
</body>
</html>
