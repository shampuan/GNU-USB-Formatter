#!/usr/bin/env python3

import sys
import subprocess
import os
import time
import tempfile
from PyQt5.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QHBoxLayout, QLabel, QComboBox,
    QLineEdit, QCheckBox, QPushButton, QMessageBox, QDialog, QFormLayout, QSizePolicy, QSpacerItem # QTextEdit kaldırıldı
)
from PyQt5.QtGui import QPixmap, QIcon, QColor, QPainter, QMovie
from PyQt5.QtCore import Qt, QThread, pyqtSignal

# Sabitler
APP_NAME = "GNU USB Formatter"
APP_VERSION = "1.0.0"
APP_LICENSE = "GNU GPL v3"
APP_AUTHOR = "@Zeus -- github.com/shampuan/"

# Dil desteği için sözlükler
TRANSLATIONS = {
    "en": {
        "about_text": f"""GNU USB Formatter
Version: {APP_VERSION}
License: {APP_LICENSE}
Author: {APP_AUTHOR}

This program formats your flash drives according to your desired settings.
This program comes with no warranty.""",
        "disk": "Disk:",
        "format": "Format:",
        "partition": "Partition:",
        "block_size": "Block size:",
        "label": "Label:",
        "quick_format": "Quick Format (Data recoverable, faster)",
        "format_button": "Format",
        "about_button": "About",
        "no_usb_found": "No USB disk found",
        "warning": "Warning",
        "no_disk_selected": "No disk selected for formatting",
        "confirmation": "Confirmation",
        "confirm_message": "<b>{}</b> disk will be formatted!\nFormat: {} | Partition: {}\nQuick Format: {}\n\nALL DATA WILL BE LOST! Continue?",
        "yes": "Yes",
        "no": "No",
        "already_running": "A process is already running",
        "success": "Success",
        "error": "Error",
        "process_started": "Process started on {} disk...",
        "admin_rights": "Requesting admin rights...",
        "format_success": "Format successful!",
        "logs_placeholder": "Process logs will be shown here...", # Bu satır kaldı, çünkü kaldırılan textedit'in placeholder'ı artık kullanılmıyor.
        "label_placeholder": "Disk label (optional)",
        "language_button": "Language",
        "unsupported_block_size": "Unsupported block size {} for {} filesystem. Using default.",
        "cluster_size_set": "Cluster size manually set to {} bytes",
        "zero_fill_started": "Disk zero-fill started. This may take a long time...",
        "zero_fill_finished": "Disk zero-fill finished.",
        "MSG": "Message"
    },
    "tr": {
        "about_text": f"""GNU USB Formatter
Versiyon: {APP_VERSION}
Lisans: {APP_LICENSE}
Yazar: {APP_AUTHOR}

Bu program, flaşbelleklerinizi istediğiniz ayarlara göre biçimlendirir.
Bu program, hiçbir garanti getirmez.""",
        "disk": "Disk:",
        "format": "Biçim:",
        "partition": "Bölümleme:",
        "block_size": "Blok Boyutu:",
        "label": "Etiket:",
        "quick_format": "Hızlı Biçimlendir (Veri kurtarılabilir, daha hızlıdır)",
        "format_button": "Biçimlendir",
        "about_button": "Hakkında",
        "no_usb_found": "USB disk bulunamadı",
        "warning": "Uyarı",
        "no_disk_selected": "Biçimlendirilecek disk seçilmedi",
        "confirmation": "Onay",
        "confirm_message": "<b>{}</b> diski biçimlendirilecek!\nBiçim: {} | Bölümleme: {}\nHızlı Biçimlendirme: {}\n\nTÜM VERİLER SİLİNECEK! Devam?",
        "yes": "Evet",
        "no": "Hayır",
        "already_running": "Zaten bir işlem çalışıyor",
        "success": "Başarılı",
        "error": "Hata",
        "process_started": "{} diski üzerinde işlem başlatılıyor...",
        "admin_rights": "Yönetici hakları isteniyor...",
        "format_success": "Biçimlendirme başarılı!",
        "logs_placeholder": "İşlem günlükleri burada gösterilecektir...", # Bu satır kaldı, çünkü kaldırılan textedit'in placeholder'ı artık kullanılmıyor.
        "label_placeholder": "Disk etiketi (isteğe bağlı)",
        "language_button": "Language",
        "unsupported_block_size": "{} dosya sistemi için {} blok boyutu desteklenmiyor. Varsayılan kullanılıyor.",
        "cluster_size_set": "Küme boyutu elle {} byte olarak ayarlandı",
        "zero_fill_started": "Diski sıfırlarla doldurma işlemi başlatıldı. Bu işlem uzun sürebilir...",
        "zero_fill_finished": "Diski sıfırlarla doldurma işlemi tamamlandı.",
        "MSG": "Mesaj"
    }
}

# Komut yolları ve desteklenen blok boyutları
COMMAND_PATHS = {
    "lsblk": "/usr/bin/lsblk",
    "mkfs.ntfs": "/usr/sbin/mkfs.ntfs",
    "mkfs.fat": "/usr/sbin/mkfs.fat",
    "mkfs.exfat": "/usr/sbin/mkfs.exfat",
    "mkfs.ext4": "/usr/sbin/mkfs.ext4",
    "parted": "/usr/sbin/parted",
    "umount": "/usr/bin/umount",
    "udevadm": "/usr/bin/udevadm",
    "sfdisk": "/usr/sbin/sfdisk",
    "dd": "/usr/bin/dd"
}

SUPPORTED_BLOCK_SIZES = {
    "NTFS": ["512", "1024", "2048", "4096", "8192", "16384", "32768", "65536"],
    "FAT32": ["512", "1024", "2048", "4096", "8192", "16384", "32768"],
    "ExFAT": ["512", "1024", "2048", "4096", "8192", "16384", "32768", "65536"],
    "Ext4": ["1024", "2048", "4096"]
}

def get_resource_path(relative_path):
    """
    Belirtilen göreceli yolu uygulamanın kaynaklarına göre bulur.
    Öncelikle betiğin çalıştığı dizine, sonra PyInstaller'ın geçici dizinine,
    son olarak da belirli sistem dizinlerine bakar.
    """
    # 1. Betiğin çalıştığı dizin
    base_path = os.path.abspath(os.path.dirname(__file__))
    current_dir_path = os.path.join(base_path, relative_path)
    if os.path.exists(current_dir_path):
        return current_dir_path

    # 2. PyInstaller tarafından oluşturulan geçici dizin (eğer varsa)
    if hasattr(sys, '_MEIPASS'):
        bundle_dir_path = os.path.join(sys._MEIPASS, relative_path)
        if os.path.exists(bundle_dir_path):
            return bundle_dir_path

    # 3. Belirli sistem dizini
    system_icon_path = os.path.join("/usr/share/GNU_USB_Formatter/icons", relative_path)
    if os.path.exists(system_icon_path):
        return system_icon_path
            
    # Hiçbir yerde bulunamazsa, orijinal göreceli yolu döndür
    # Bu, PyQt'nin kendi varsayılanlarını kullanmasına veya hata vermesine neden olabilir
    return relative_path


class DiskOperationThread(QThread):
    finished = pyqtSignal(bool, str)
    # progress = pyqtSignal(str) # Bu satır kaldırıldı çünkü artık hiçbir widget'a ilerleme bilgisi gönderilmiyor.

    def __init__(self, disk, format_type, block_size, label, quick_format, partition_scheme="msdos", language="tr"):
        super().__init__()
        self.disk = disk
        self.format_type = format_type
        self.block_size = block_size
        self.label = label
        self.quick_format = quick_format
        self.partition_scheme = partition_scheme
        self.language = language

    def tr(self, key):
        return TRANSLATIONS[self.language].get(key, key)

    def run(self):
        temp_script_path = None
        try:
            # self.progress.emit(self.tr("process_started").format(self.disk)) # Kaldırıldı

            script_content = []
            script_content.append("#!/bin/bash")
            script_content.append("set -e")

            # Tüm bağlı bölümleri çıkar
            script_content.append(f"{COMMAND_PATHS.get('lsblk')} -o NAME,MOUNTPOINT -nl -p {self.disk} | awk '$2 != \"\" {{print $1}}' | while read -r part; do")
            script_content.append(f"    {COMMAND_PATHS.get('umount')} \"$part\" || true")
            script_content.append("done")

            # Eğer hızlı biçimlendirme seçili değilse, diski sıfırlarla doldur
            if not self.quick_format:
                dd_path = COMMAND_PATHS.get("dd")
                if not dd_path:
                    raise FileNotFoundError(f"dd {self.tr('komutu bulunamadı')}")
                
                # script_content.append(f"echo \"{self.tr('zero_fill_started')}\"") # Kaldırıldı
                # dd komutu stderr'e progress bilgisi basar, ancak artık GUI'ye gönderilmeyecek
                script_content.append(f"{dd_path} if=/dev/zero of=\"{self.disk}\" bs=4M status=progress oflag=sync || true") 
                # script_content.append(f"echo \"{self.tr('zero_fill_finished')}\"") # Kaldırıldı
            
            # Bölüm tablosu oluştur
            script_content.append(f"echo \"{self.partition_scheme} bölüm tablosu oluşturuluyor...\"")
            script_content.append(f"{COMMAND_PATHS.get('parted')} -s {self.disk} mklabel {self.partition_scheme}")
            
            # Birincil bölüm oluştur
            script_content.append(f"echo \"Birincil bölüm oluşturuluyor...\"")
            script_content.append(f"{COMMAND_PATHS.get('parted')} -s {self.disk} mkpart primary 0% 100%")
            
            # Bölüm türünü format tipine göre ayarla
            script_content.append(f"echo \"Bölüm türü ayarlanıyor...\"")
            if self.partition_scheme == "msdos":  # MBR için
                if self.format_type == "NTFS":
                    script_content.append(f"echo -e 'type=7' | {COMMAND_PATHS.get('sfdisk')} --label dos {self.disk}")
                elif self.format_type == "FAT32":
                    script_content.append(f"echo -e 'type=c' | {COMMAND_PATHS.get('sfdisk')} --label dos {self.disk}")
                elif self.format_type == "ExFAT":
                    script_content.append(f"echo -e 'type=7' | {COMMAND_PATHS.get('sfdisk')} --label dos {self.disk}")
                elif self.format_type == "Ext4":
                    script_content.append(f"echo -e 'type=83' | {COMMAND_PATHS.get('sfdisk')} --label dos {self.disk}")
            else:  # GPT için
                if self.format_type == "NTFS":
                    script_content.append(f"{COMMAND_PATHS.get('parted')} -s {self.disk} set 1 msftdata on")
                elif self.format_type == "FAT32":
                    script_content.append(f"{COMMAND_PATHS.get('parted')} -s {self.disk} set 1 esp on")
                elif self.format_type == "ExFAT":
                    script_content.append(f"{COMMAND_PATHS.get('parted')} -s {self.disk} set 1 msftdata on")

            script_content.append(f"{COMMAND_PATHS.get('udevadm')} settle")
            script_content.append("sleep 2")

            # Bölüm adını belirle (nvme vs normal diskler için)
            script_content.append("if [[ \"{self.disk}\" == *\"nvme\"* ]]; then")
            script_content.append(f"    PARTITION=\"{self.disk}p1\"")
            script_content.append("else")
            script_content.append(f"    PARTITION=\"{self.disk}1\"")
            script_content.append("fi")
            script_content.append("echo \"Bölüm: $PARTITION\"")

            # Formatlama komutunu oluştur
            mkfs_command = []
            mkfs_cmd_name = {
                "NTFS": "mkfs.ntfs",
                "FAT32": "mkfs.fat",
                "ExFAT": "mkfs.exfat",
                "Ext4": "mkfs.ext4"
            }.get(self.format_type)

            if not mkfs_cmd_name:
                raise Exception(self.tr("Geçersiz dosya sistemi seçimi."))
            
            mkfs_path = COMMAND_PATHS.get(mkfs_cmd_name)
            if not mkfs_path:
                raise FileNotFoundError(f"{mkfs_cmd_name} {self.tr('komutu bulunamadı')}")

            mkfs_command.append(mkfs_path)

            # Blok boyutu ayarı (NTFS için özel zorlama parametreleri)
            if self.block_size != "Varsayılan":
                if self.format_type == "NTFS":
                    mkfs_command.extend(['-c', self.block_size, '--force', '-f'])
                    # self.progress.emit(self.tr("cluster_size_set").format(self.block_size)) # Kaldırıldı
                elif self.format_type == "FAT32":
                    mkfs_command.extend(['-S', self.block_size])
                    # self.progress.emit(self.tr("cluster_size_set").format(self.block_size)) # Kaldırıldı
                elif self.format_type == "ExFAT":
                    mkfs_command.extend(['-c', self.block_size])
                    # self.progress.emit(self.tr("cluster_size_set").format(self.block_size)) # Kaldırıldı
                elif self.format_type == "Ext4":
                    mkfs_command.extend(['-b', self.block_size])
                    # self.progress.emit(self.tr("cluster_size_set").format(self.block_size)) # Kaldırıldı

            if self.format_type == "NTFS":
                if self.quick_format:
                    mkfs_command.append('-Q')
                if self.label:
                    mkfs_command.extend(['-L', self.label])
            elif self.format_type == "FAT32":
                mkfs_command.extend(['-F', '32'])
                if not self.quick_format:
                    mkfs_command.append('-c')
                if self.label:
                    mkfs_command.extend(['-n', self.label[:11]])
            elif self.format_type == "Ext4":
                if not self.quick_format:
                    mkfs_command.append('-c')
                if self.label:
                    mkfs_command.extend(['-L', self.label])
            elif self.format_type == "ExFAT":
                if self.label:
                    mkfs_command.extend(['-L', self.label])

            mkfs_command.append("$PARTITION")
            # script_content.append(f"echo \"{self.tr('Biçimlendirme başlıyor:')} {' '.join(mkfs_command)}\"") # Kaldırıldı
            script_content.append(' '.join(mkfs_command))

            with tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.sh', encoding='utf-8') as tmp_file:
                tmp_file.write('\n'.join(script_content))
                temp_script_path = tmp_file.name
            
            os.chmod(temp_script_path, 0o755)
            # self.progress.emit(self.tr("admin_rights")) # Kaldırıldı
            
            proc = subprocess.Popen(['pkexec', temp_script_path],
                                  stdout=subprocess.PIPE,
                                  stderr=subprocess.PIPE,
                                  text=True)
            
            # dd komutunun ilerleme çıktısı stderr'den gelebilir, ancak artık GUI'ye gönderilmeyecek
            while True:
                line_stdout = proc.stdout.readline()
                line_stderr = proc.stderr.readline()
                if not line_stdout and not line_stderr and proc.poll() is not None:
                    break
                # if line_stdout: # Kaldırıldı
                #     pass
                # if line_stderr: # Kaldırıldı
                #     pass

            proc.wait()
            
            if proc.returncode != 0:
                raise Exception(f"{self.tr('İşlem hatası')} (Kod: {proc.returncode})")
            
            self.finished.emit(True, self.tr("format_success"))

        except Exception as e:
            self.finished.emit(False, f"{self.tr('error')}: {str(e)}")
        finally:
            if temp_script_path and os.path.exists(temp_script_path):
                try:
                    os.remove(temp_script_path)
                except:
                    pass

class USBFormatterApp(QWidget):
    def __init__(self):
        super().__init__()
        self.current_language = "tr"
        self.setWindowTitle(APP_NAME)
        # Resim yolunu get_resource_path ile bul
        self.setWindowIcon(QIcon(get_resource_path('usb_icon.png')))
        # QTextEdit kaldırıldığı için pencere yüksekliği ayarlandı.
        self.setGeometry(100, 100, 400, 400) 
        self.thread = None
        
        # Animasyon için QMovie nesnesi (yolu get_resource_path ile bulacak)
        self.movie = QMovie(get_resource_path('usb_icon.gif'))
        self.usb_icon_label = None
        self.original_pixmap = None
        
        self.init_ui()
        self.load_disks()

    def tr(self, key):
        return TRANSLATIONS[self.current_language].get(key, key)

    def init_ui(self):
        main_layout = QVBoxLayout()
        main_layout.setContentsMargins(20, 20, 20, 20)
        main_layout.setSpacing(15)
        
        # Resim için QLabel oluştur
        self.usb_icon_label = QLabel()
        self.usb_icon_label.setAlignment(Qt.AlignCenter)
        self.set_static_icon()
        
        # Resim için bir container layout oluşturarak daha iyi merkezleme sağlıyoruz
        icon_container = QHBoxLayout()
        icon_container.addStretch()
        icon_container.addWidget(self.usb_icon_label)
        icon_container.addStretch()
        
        main_layout.addLayout(icon_container)
        
        # Resim ile form arasında boşluk ekle
        main_layout.addSpacing(10)

        form_layout = QFormLayout()
        form_layout.setVerticalSpacing(10)
        
        self.disk_label = QLabel(self.tr("disk"))
        self.disk_combo = QComboBox()
        form_layout.addRow(self.disk_label, self.disk_combo)

        self.format_label = QLabel(self.tr("format"))
        self.format_combo = QComboBox()
        self.format_combo.addItems(["NTFS", "FAT32", "ExFAT", "Ext4"])
        self.format_combo.currentTextChanged.connect(self.update_block_size_options)
        form_layout.addRow(self.format_label, self.format_combo)

        self.partition_label = QLabel(self.tr("partition"))
        self.partition_combo = QComboBox()
        self.partition_combo.addItems(["MBR", "GPT"])
        form_layout.addRow(self.partition_label, self.partition_combo)

        self.block_size_label = QLabel(self.tr("block_size"))
        self.block_size_combo = QComboBox()
        self.update_block_size_options()
        form_layout.addRow(self.block_size_label, self.block_size_combo)

        self.label_label = QLabel(self.tr("label"))
        self.label_input = QLineEdit()
        self.label_input.setPlaceholderText(self.tr("label_placeholder"))
        form_layout.addRow(self.label_label, self.label_input)

        main_layout.addLayout(form_layout)

        self.quick_format_check = QCheckBox(self.tr("quick_format"))
        self.quick_format_check.setChecked(True)
        main_layout.addWidget(self.quick_format_check)

        button_layout = QHBoxLayout()
        
        self.format_button = QPushButton(self.tr("format_button"))
        self.format_button.clicked.connect(self.start_format_process)
        button_layout.addWidget(self.format_button)

        self.language_button = QPushButton(self.tr("language_button"))
        self.language_button.clicked.connect(self.toggle_language)
        button_layout.addWidget(self.language_button)

        main_layout.addLayout(button_layout)

        self.about_button = QPushButton(self.tr("about_button"))
        self.about_button.clicked.connect(self.show_about_dialog)
        main_layout.addWidget(self.about_button)
        
        # BURASI KALDIRILDI!!!
        # self.status_log = QTextEdit() 
        # self.status_log.setReadOnly(True) 
        # self.status_log.setPlaceholderText(self.tr("logs_placeholder")) 
        # main_layout.addWidget(self.status_log) 

        self.setLayout(main_layout)

    def update_block_size_options(self):
        current_format = self.format_combo.currentText()
        self.block_size_combo.clear()
        self.block_size_combo.addItem("Varsayılan")
        self.block_size_combo.addItems(SUPPORTED_BLOCK_SIZES.get(current_format, []))

    def set_static_icon(self):
        """Statik PNG resmini göster"""
        if hasattr(self, 'movie') and self.movie:
            self.movie.stop()
        
        # Resim yolunu get_resource_path ile bul
        pixmap = QPixmap(get_resource_path('usb_icon.png'))
        if pixmap.isNull():
            # Eğer resim bulunamazsa varsayılan bir daire çiz
            pixmap = QPixmap(100, 100)
            pixmap.fill(Qt.transparent)
            painter = QPainter(pixmap)
            painter.setRenderHint(QPainter.Antialiasing)
            painter.setBrush(QColor(102, 51, 153))
            painter.drawEllipse(0, 0, 100, 100)
            painter.end()
        
        # Orjinal boyutları kaydet
        self.original_pixmap = pixmap.scaled(100, 100, Qt.KeepAspectRatio)
        self.usb_icon_label.setPixmap(self.original_pixmap)
        self.usb_icon_label.setFixedSize(100, 100)

    def set_animated_icon(self):
        """Animasyonlu GIF'i göster ve boyutunu ayarla"""
        gif_path = get_resource_path('usb_icon.gif')
        if os.path.exists(gif_path):
            self.movie = QMovie(gif_path)
            
            # Boyut ayarını doğru yap
            self.movie.setScaledSize(self.original_pixmap.size())
            self.usb_icon_label.setMovie(self.movie)
            self.usb_icon_label.setFixedSize(self.original_pixmap.size())
            
            if self.movie.isValid():
                self.movie.start()
        else:
            self.set_static_icon()

    def toggle_language(self):
        """Dil değiştirme fonksiyonu"""
        self.current_language = "en" if self.current_language == "tr" else "tr"
        self.update_ui_language()

    def update_ui_language(self):
        """Arayüz dilini günceller"""
        self.setWindowTitle(APP_NAME)
        
        self.disk_label.setText(self.tr("disk"))
        self.format_label.setText(self.tr("format"))
        self.partition_label.setText(self.tr("partition"))
        self.block_size_label.setText(self.tr("block_size"))
        self.label_label.setText(self.tr("label"))
        
        self.quick_format_check.setText(self.tr("quick_format"))
        self.format_button.setText(self.tr("format_button"))
        self.about_button.setText(self.tr("about_button"))
        self.language_button.setText(self.tr("language_button"))
        # self.status_log.setPlaceholderText(self.tr("logs_placeholder")) # Kaldırıldı
        self.label_input.setPlaceholderText(self.tr("label_placeholder"))
        
        # Disk listesini yenile
        self.load_disks()

    def load_disks(self):
        self.disk_combo.clear()
        try:
            result = subprocess.run(
                [COMMAND_PATHS['lsblk'], '-o', 'NAME,RM,TYPE,SIZE,MODEL', '-nl', '-p'],
                capture_output=True, text=True)
            
            disks = []
            for line in result.stdout.splitlines():
                parts = line.split()
                if len(parts) >= 4 and parts[2] == 'disk' and parts[1] == '1':
                    disk_info = f"{parts[0]} ({parts[3]})"
                    if len(parts) > 4:
                        disk_info += f" [{' '.join(parts[4:])}]"
                    disks.append(disk_info)
            
            if disks:
                self.disk_combo.addItems(disks)
            else:
                self.disk_combo.addItem(self.tr("no_usb_found"))
                
        except Exception as e:
            # Hata mesajı artık doğrudan bir MessageBox ile gösteriliyor.
            QMessageBox.critical(self, self.tr("error"), f"{self.tr('error')}: {str(e)}")

    def start_format_process(self):
        selected_disk_text = self.disk_combo.currentText()
        if not selected_disk_text or self.tr("no_usb_found") in selected_disk_text:
            QMessageBox.warning(self, self.tr("warning"), self.tr("no_disk_selected"))
            return

        disk_path = selected_disk_text.split()[0]
        format_type = self.format_combo.currentText()
        partition_scheme = "msdos" if "MBR" in self.partition_combo.currentText() else "gpt"
        quick_format = self.quick_format_check.isChecked()
        block_size = self.block_size_combo.currentText()
        
        reply = QMessageBox.warning(
            self, self.tr("confirmation"),
            self.tr("confirm_message").format(
                disk_path, 
                format_type, 
                partition_scheme,
                self.tr("yes") if quick_format else self.tr("no")
            ),
            QMessageBox.Yes | QMessageBox.No
        )
        
        if reply == QMessageBox.Yes:
            # self.status_log.clear() # Kaldırıldı
            self.start_format_thread(
                disk_path,
                format_type,
                block_size,
                self.label_input.text().strip(),
                quick_format,
                partition_scheme
            )

    def start_format_thread(self, disk, format_type, block_size, label, quick_format, partition_scheme):
        if self.thread and self.thread.isRunning():
            QMessageBox.warning(self, self.tr("warning"), self.tr("already_running"))
            return

        self.format_button.setEnabled(False)
        self.set_animated_icon()  # İşlem başladığında animasyonu başlat
        
        self.thread = DiskOperationThread(
            disk=disk,
            format_type=format_type,
            block_size=block_size,
            label=label,
            quick_format=quick_format,
            partition_scheme=partition_scheme,
            language=self.current_language
        )
        self.thread.finished.connect(self.format_finished)
        # self.thread.progress.connect(self.update_status_log) # Kaldırıldı
        self.thread.start()

    def format_finished(self, success, message):
        self.format_button.setEnabled(True)
        self.set_static_icon()  # İşlem bittiğinde statik resme dön
        
        if success:
            QMessageBox.information(self, self.tr("success"), message)
            self.load_disks()
        else:
            QMessageBox.critical(self, self.tr("error"), message)

    def update_status_log(self, message):
        pass # Bu fonksiyonun içi boşaltıldı ve çağrısı da kaldırıldı.

    def show_about_dialog(self):
        about_dialog = QMessageBox(self)
        about_dialog.setWindowTitle(self.tr("about_button"))
        about_dialog.setText(self.tr("about_text"))
        # Resim yolunu get_resource_path ile bul
        about_dialog.setIconPixmap(QPixmap(get_resource_path('usb_icon.png')).scaled(64, 64))
        about_dialog.exec_()

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = USBFormatterApp()
    ex.show()
    sys.exit(app.exec_())
