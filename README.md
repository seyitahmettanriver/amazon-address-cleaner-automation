#### Amazon Address Cleaner Automation

**Description:**  
This script automates the process of deleting old or unwanted addresses on Amazon. By providing a control panel, users can start or stop the automation with a single click. When active, the script will automatically delete addresses from the account, making address management much easier.

---

### Features:
- **Automatic Address Deletion**: Deletes old addresses from Amazon accounts automatically.
- **Control Panel**: A floating control panel to start/stop the automation.
- **Start/Stop Button**: Easily toggle the automation on or off.
- **Developer Info**: Contact information for the developer.
- **Simple User Interface**: The control panel is user-friendly and positioned at the top right of the page for easy access.

---

### How to Use:

1. **Installation**:
   - Install the **Tampermonkey** extension in your browser.
   - Create a new script in Tampermonkey and paste the code provided below into the script editor.
   - Save and activate the script.

2. **Control Panel**:
   - A panel will appear on the Amazon addresses page (`https://www.amazon.com/a/addresses*`).
   - Use the **Start** button to enable the automatic address deletion.
   - The **Stop** button will pause the process, allowing you to review your addresses.

3. **Automation**:
   - When the automation is active, the script will go through all addresses, and it will delete them one by one.
   - Once all addresses are deleted, the page will automatically reload.

---

### Configuration:
- **Toggle Automation**: The script uses `localStorage` to keep track of whether the automation is active or not. This state is saved even after refreshing the page.
- **CSRF Token**: The script retrieves the CSRF token from the Amazon address page to authorize the delete requests.

---

### Developer Information:
- **Author**: [Seyit Ahmet TANRIVER](https://wa.me/31637952159)

---

### License:
This project is open-source and available under the MIT License.

---

### Contact & Support:
- For any questions or support, feel free to open an issue on GitHub or contact the developer directly at [Seyit Ahmet TANRIVER](https://wa.me/31637952159).

---

### Full Script Code:

```javascript
// ==UserScript==
// @name         Amazon Adres Temizleme Otomasyonu
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Amazon üzerindeki eski adresleri otomatik olarak silme işlemi.
// @author       Seyit Ahmet TANRIVER
// @match        https://www.amazon.com/a/addresses*
// @grant        none
// @require      https://code.jquery.com/jquery-3.6.4.min.js
// ==/UserScript==

(function () {
    'use strict';

    // Paneli oluştur
    function createControlPanel() {
        if (!document.getElementById('control-panel')) {
            const panel = document.createElement('div');
            panel.id = 'control-panel';
            panel.style.cssText = `
                position: fixed;
                top: 20px;
                right: 20px;
                z-index: 9999;
                background: #ffffff;
                border: 2px solid #007bff;
                border-radius: 12px;
                padding: 15px;
                box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
                font-family: 'Arial', sans-serif;
                width: 250px;
                animation: slideIn 0.5s ease;
            `;

            // Animasyon
            const style = document.createElement('style');
            style.innerHTML = `
                @keyframes slideIn {
                    from { opacity: 0; transform: translateX(20px); }
                    to { opacity: 1; transform: translateX(0); }
                }
            `;
            document.head.appendChild(style);

            // Panel başlığı
            const header = document.createElement('div');
            header.style.cssText = `
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-bottom: 15px;
            `;
            panel.appendChild(header);

            const title = document.createElement('h4');
            title.textContent = 'Adres Silme Paneli';
            title.style.cssText = `
                margin: 0;
                font-size: 18px;
                color: #007bff;
            `;
            header.appendChild(title);

            // Kapatma butonu
            const closeButton = document.createElement('button');
            closeButton.innerHTML = '&#10006;'; // Unicode çarpı işareti
            closeButton.style.cssText = `
                background: none;
                border: none;
                font-size: 18px;
                color: #ff4d4f;
                cursor: pointer;
                transition: color 0.3s;
            `;
            closeButton.addEventListener('mouseenter', () => {
                closeButton.style.color = '#ff1a1a';
            });
            closeButton.addEventListener('mouseleave', () => {
                closeButton.style.color = '#ff4d4f';
            });
            closeButton.addEventListener('click', () => {
                panel.remove();
            });
            header.appendChild(closeButton);

            // Başlat/Durdur butonu
            const toggleButton = document.createElement('button');
            toggleButton.id = 'toggle-button';
            toggleButton.style.cssText = `
                color: white;
                border: none;
                padding: 10px;
                width: 100%;
                border-radius: 8px;
                cursor: pointer;
                font-size: 16px;
                font-weight: bold;
                box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
                transition: background-color 0.3s;
            `;
            updateButtonState(toggleButton); // Buton durumunu ayarla
            toggleButton.addEventListener('click', () => {
                toggleAutomation(toggleButton); // Otomasyonu başlat/durdur
            });
            panel.appendChild(toggleButton);

            // Geliştirici bilgisi
            const developerInfo = document.createElement('p');
            developerInfo.innerHTML = `
                Geliştirici:
                <a href="https://wa.me/31637952159"
                   style="color: #007bff; text-decoration: none;"
                   target="_blank">
                   Seyit Ahmet TANRIVER
                </a>
            `;
            developerInfo.style.cssText = `
                font-size: 12px;
                color: #666;
                text-align: center;
                margin-top: 15px;
            `;
            panel.appendChild(developerInfo);

            // Paneli sayfaya ekle
            document.body.appendChild(panel);
        }
    }

    // Otomasyonu başlat veya durdur
    function toggleAutomation(button) {
        const isActive = localStorage.getItem('deleteAutomationActive') === 'true';
        if (isActive) {
            localStorage.setItem('deleteAutomationActive', 'false');
            updateButtonState(button); // Durumu güncelle
            alert('Otomasyon durduruldu.');
        } else {
            localStorage.setItem('deleteAutomationActive', 'true');
            updateButtonState(button); // Durumu güncelle
            alert('Otomasyon başlatıldı.');
            processDeletion();
        }
    }

    // Buton durumunu güncelle
    function updateButtonState(button) {
        const isActive = localStorage.getItem('deleteAutomationActive') === 'true';
        if (isActive) {
            button.textContent = 'Durdur';
            button.style.background = '#f44336'; // Kırmızı
        } else {
            button.textContent = 'Başlat';
            button.style.background = '#4caf50'; // Yeşil
        }
    }

    // Silme işlemini yap
    function processDeletion() {
        if (localStorage.getItem('deleteAutomationActive') !== 'true') {
            console.log('Otomasyon durduruldu.');
            return;
        }

        const addressInputs = $('.address-column').find('input[name="addressID"]');
        if (addressInputs.length > 0) {
            $('.address-column').each(function () {
                const addressColumn = $(this);
                const addressID = addressColumn.find('input[name="addressID"]').val();
                const csrfToken = $('input[name="csrfToken"]:first').val();

                if (addressID && csrfToken) {
                    $.post('/a/addresses/delete', {
                        'addressID': addressID,
                        'isStoreAddress': 'false',
                        'csrfToken': csrfToken
                    }).done(function () {
                        addressColumn.find('.a-box').css('background-color', '#ef4444').fadeOut(300, function () {
                            addressColumn.remove();
                            if ($('.address-column').find('input[name="addressID"]').length === 0) {
                                location.reload(); // Sayfayı yenile
                            }
                        });
                    }).fail(function (xhr, status, error) {
                        console.error('Ajax isteği sırasında bir hata oluştu:', error);
                    });
                }
            });
        } else {
            console.log('Tüm adresler silindi, işlem tamamlandı.');
            localStorage.setItem('deleteAutomationActive', 'false');
            updateButtonState(document.getElementById('toggle-button'));
        }
    }

    // Sayfa yüklendiğinde kontrol et
    window.addEventListener('load', () => {
        createControlPanel();
        if (localStorage.getItem('deleteAutomationActive') === 'true') {
            processDeletion();
        }
    });
})();
```

---

Feel free to create issues or contribute to the project.
