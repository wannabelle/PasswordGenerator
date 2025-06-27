import sys
import string
import random
import json
import os
from datetime import datetime
from PyQt5.QtWidgets import (
    QApplication, QWidget, QLabel, QVBoxLayout, QHBoxLayout,
    QPushButton, QSpinBox, QCheckBox, QLineEdit, QSizePolicy,
    QMessageBox, QTabWidget, QTableWidget, QTableWidgetItem,
    QHeaderView, QAbstractItemView, QFrame, QScrollArea, QComboBox
)
from PyQt5.QtGui import QFont
from PyQt5.QtCore import Qt

class PasswordGeneratorApp(QWidget):
    DATA_FILE = "password_history.json"
    MAX_CONTAINER_WIDTH = 1200

    def __init__(self):
        super().__init__()
        self.setWindowTitle("Password Generator")
        self.resize(1024, 700)
        self.password_history = []
        self.current_style = self.default_style()
        self.setup_ui()
        self.apply_styles()
        self.load_history()

    def default_style(self):
        return {
            "background": "#ffffff",
            "text": "#6b7280",
            "header": "#111827",
            "subheader": "#4b5563",
            "button_bg": "#111827",
            "button_text": "white",
            "input_bg": "#f9fafb",
            "input_text": "#111827",
            "table_bg": "#f9fafb",
            "table_header": "#e5e7eb",
            "table_item": "#374151",
            "selected_bg": "#2563eb",
            "selected_text": "white",
            "tab_bg": "#f3f4f6",
            "tab_text": "#6b7280",
            "tab_selected_bg": "#111827",
            "tab_selected_text": "white",
            "checkbox_text": "#374151"
        }

    def apply_styles(self):
        s = self.current_style
        self.setStyleSheet(f"""
            QWidget {{
                background-color: {s['background']};
                color: {s['text']};
                font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
                font-size: 17px;
            }}
            QLabel#headerLabel {{
                font-size: 48px;
                font-weight: 700;
                color: {s['header']};
                margin-bottom: 30px;
            }}
            QLabel#subHeaderLabel {{
                font-size: 18px;
                color: {s['subheader']};
                margin-bottom: 36px;
            }}
            QSpinBox {{
                font-size: 16px;
                padding: 6px 12px;
                border: 1px solid #d1d5db;
                border-radius: 12px;
                min-width: 90px;
                color: #374151;
            }}
            QCheckBox {{
                font-size: 16px;
                padding: 8px 0;
                color: {s['checkbox_text']};
            }}
            QPushButton#generateButton {{
                background-color: {s['button_bg']};
                color: {s['button_text']};
                font-weight: 700;
                font-size: 20px;
                border-radius: 14px;
                padding: 16px 0;
                margin-top: 24px;
                transition: background-color 0.3s ease;
                letter-spacing: 1.2px;
            }}
            QPushButton#generateButton:hover {{
                background-color: #374151;
            }}
            QLineEdit#outputPassword {{
                border: 1px solid #d1d5db;
                border-radius: 14px;
                padding: 14px 20px;
                font-size: 20px;
                font-weight: 600;
                letter-spacing: 4px;
                background-color: {s['input_bg']};
                color: {s['input_text']};
                selection-background-color: #2563eb;
                selection-color: white;
            }}
            QTableWidget {{
                border: none;
                border-radius: 14px;
                background-color: {s['table_bg']};
                font-size: 16px;
            }}
            QHeaderView::section {{
                background-color: {s['table_header']};
                padding: 8px;
                border: none;
                font-weight: 700;
                color: {s['table_item']};
            }}
            QTableWidget::item {{
                padding: 10px 14px;
                border-bottom: 1px solid #e5e7eb;
                color: {s['table_item']};
            }}
            QTableWidget::item:selected {{
                background-color: {s['selected_bg']};
                color: {s['selected_text']};
            }}
            QWidget#cardContainer {{
                background-color: {s['input_bg']};
                border-radius: 24px;
                padding: 48px 56px;
                max-width: {self.MAX_CONTAINER_WIDTH}px;
                margin-left: auto;
                margin-right: auto;
                box-shadow: 0 10px 40px rgba(17, 24, 39, 0.07);
            }}
            QTabWidget::pane {{
                border: none;
                margin: 8px;
                background: {s['tab_bg']};
                border-radius: 12px;
            }}
            QTabBar::tab {{
                background: {s['tab_bg']};
                border-radius: 12px;
                padding: 12px 32px;
                font-weight: 600;
                color: {s['tab_text']};
                min-width: 140px;
                margin-right: 12px;
                transition: background-color 0.3s ease, color 0.3s ease;
            }}
            QTabBar::tab:selected {{
                background: {s['tab_selected_bg']};
                color: {s['tab_selected_text']};
            }}
        """)

    def setup_ui(self):
        mainLayout = QVBoxLayout(self)
        mainLayout.setAlignment(Qt.AlignTop | Qt.AlignHCenter)
        mainLayout.setContentsMargins(0, 0, 0, 0)

        scrollArea = QScrollArea()
        scrollArea.setWidgetResizable(True)
        mainLayout.addWidget(scrollArea)

        container = QWidget()
        container.setObjectName("cardContainer")
        scrollArea.setWidget(container)

        containerLayout = QVBoxLayout(container)
        containerLayout.setContentsMargins(48, 36, 48, 36)
        containerLayout.setSpacing(28)

        header = QLabel("Password Generator")
        header.setObjectName("headerLabel")
        header.setAlignment(Qt.AlignCenter)
        containerLayout.addWidget(header)

        subheader = QLabel("Create strong, secure passwords instantly tailored to your needs.")
        subheader.setObjectName("subHeaderLabel")
        subheader.setAlignment(Qt.AlignCenter)
        containerLayout.addWidget(subheader)

        self.tabs = QTabWidget()
        self.tabs.setDocumentMode(True)
        self.tabs.setMovable(False)
        containerLayout.addWidget(self.tabs)

        self.tab_generate = QWidget()
        self.tab_history = QWidget()
        self.tab_preset_color = QWidget()

        self.tabs.addTab(self.tab_generate, "Generate")
        self.tabs.addTab(self.tab_history, "History")
        self.tabs.addTab(self.tab_preset_color, "Preset Color")

        self._setup_generate_tab()
        self._setup_history_tab()
        self._setup_preset_color_tab()

    def _setup_generate_tab(self):
        layout = QVBoxLayout(self.tab_generate)
        layout.setSpacing(20)

        lenLayout = QHBoxLayout()
        lenLabel = QLabel("Password Length:")
        lenLabel.setMinimumWidth(180)
        lenLabel.setFont(QFont("Segoe UI", 18, QFont.Bold))
        lenLayout.addWidget(lenLabel)

        self.lengthSpin = QSpinBox()
        self.lengthSpin.setRange(6, 128)  # original range with minimum 6
        self.lengthSpin.setValue(16)
        self.lengthSpin.setFont(QFont("Segoe UI", 18))
        self.lengthSpin.setMinimumWidth(90)
        lenLayout.addWidget(self.lengthSpin)

        lenLayout.addStretch()
        layout.addLayout(lenLayout)

        self.chkUpper = QCheckBox("Include Uppercase (A-Z)")
        self.chkUpper.setChecked(True)
        self.chkUpper.setFont(QFont("Segoe UI", 16))

        self.chkLower = QCheckBox("Include Lowercase (a-z)")
        self.chkLower.setChecked(True)
        self.chkLower.setFont(QFont("Segoe UI", 16))

        self.chkDigits = QCheckBox("Include Digits (0-9)")
        self.chkDigits.setChecked(True)
        self.chkDigits.setFont(QFont("Segoe UI", 16))

        self.chkSymbols = QCheckBox("Include Symbols (!@#$...)")
        self.chkSymbols.setChecked(False)
        self.chkSymbols.setFont(QFont("Segoe UI", 16))

        layout.addWidget(self.chkUpper)
        layout.addWidget(self.chkLower)
        layout.addWidget(self.chkDigits)
        layout.addWidget(self.chkSymbols)

        self.btnGenerate = QPushButton("Generate Password")
        self.btnGenerate.setObjectName("generateButton")
        self.btnGenerate.setFont(QFont("Segoe UI", 20, QFont.Bold))
        self.btnGenerate.clicked.connect(self.generate_password)
        layout.addWidget(self.btnGenerate)

        outputLayout = QHBoxLayout()
        self.outputPassword = QLineEdit()
        self.outputPassword.setReadOnly(True)
        self.outputPassword.setObjectName("outputPassword")
        self.outputPassword.setPlaceholderText("Your generated password will appear here")
        self.outputPassword.setFont(QFont("Segoe UI", 18, QFont.DemiBold))
        outputLayout.addWidget(self.outputPassword)

        self.btnCopy = QPushButton("Copy")
        self.btnCopy.setObjectName("copyButton")
        self.btnCopy.setFont(QFont("Segoe UI", 16, QFont.Bold))
        self.btnCopy.setSizePolicy(QSizePolicy.Fixed, QSizePolicy.Fixed)
        self.btnCopy.clicked.connect(self.copy_password)
        outputLayout.addWidget(self.btnCopy)

        layout.addLayout(outputLayout)

    def generate_password(self):
        length = self.lengthSpin.value()
        char_sets = []
        if self.chkUpper.isChecked():
            char_sets.append(string.ascii_uppercase)
        if self.chkLower.isChecked():
            char_sets.append(string.ascii_lowercase)
        if self.chkDigits.isChecked():
            char_sets.append(string.digits)
        if self.chkSymbols.isChecked():
            char_sets.append('!@#$%^&*()-_=+[]{}|;:,.<>?/~`')

        if not char_sets:
            QMessageBox.warning(self, "Warning", "Please select at least one character set.")
            return

        password_chars = [random.choice(char_set) for char_set in char_sets]
        all_chars = ''.join(char_sets)
        password_chars += [random.choice(all_chars) for _ in range(length - len(password_chars))]
        random.shuffle(password_chars)
        password = ''.join(password_chars)
        self.outputPassword.setText(password)

        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.password_history.append((password, timestamp))
        self._add_history_entry(password, timestamp)
        self.save_history()

    def _add_history_entry(self, password, timestamp):
        row = self.historyTable.rowCount()
        self.historyTable.insertRow(row)
        self.historyTable.setItem(row, 0, QTableWidgetItem(password))
        self.historyTable.setItem(row, 1, QTableWidgetItem(timestamp))
        self.historyTable.setItem(row, 2, QTableWidgetItem(str(len(password))))
        self.historyTable.item(row, 0).setToolTip(password)

    def copy_password(self):
        password = self.outputPassword.text()
        if password:
            QApplication.clipboard().setText(password)
            QMessageBox.information(self, "Copied!", "Password copied to clipboard.")
        else:
            QMessageBox.warning(self, "Warning", "No password to copy.")

    def copy_selected_history(self):
        selected_rows = self.historyTable.selectionModel().selectedRows()
        if not selected_rows:
            QMessageBox.warning(self, "Warning", "Please select a password from history to copy.")
            return
        row = selected_rows[0].row()
        password = self.historyTable.item(row, 0).text()
        QApplication.clipboard().setText(password)
        QMessageBox.information(self, "Copied!", "Selected password copied to clipboard.")

    def delete_selected_history(self):
        selected_rows = self.historyTable.selectionModel().selectedRows()
        if not selected_rows:
            QMessageBox.warning(self, "Warning", "Please select a password from history to delete.")
            return
        confirm = QMessageBox.question(self, "Confirm Delete",
                                       "Are you sure you want to delete the selected password(s)?",
                                       QMessageBox.Yes | QMessageBox.No)
        if confirm != QMessageBox.Yes:
            return
        rows = sorted([r.row() for r in selected_rows], reverse=True)
        for row in rows:
            del self.password_history[row]
            self.historyTable.removeRow(row)
        self.save_history()

    def save_history(self):
        try:
            with open(self.DATA_FILE, 'w', encoding='utf-8') as f:
                json.dump(self.password_history, f, ensure_ascii=False, indent=4)
        except Exception as e:
            QMessageBox.critical(self, "Error", f"Failed to save history:\n{e}")

    def load_history(self):
        if os.path.exists(self.DATA_FILE):
            try:
                with open(self.DATA_FILE, 'r', encoding='utf-8') as f:
                    self.password_history = json.load(f)
                    for entry in self.password_history:
                        if len(entry) == 2:
                            self._add_history_entry(entry[0], entry[1])
            except Exception as e:
                QMessageBox.warning(self, "Warning", f"Failed to load history:\n{e}")
                self.password_history = []
        else:
            self.password_history = []

    def _setup_history_tab(self):
        layout = QVBoxLayout(self.tab_history)
        layout.setContentsMargins(0, 0, 0, 0)

        self.historyTable = QTableWidget(0, 3)
        self.historyTable.setHorizontalHeaderLabels(["Password", "Generated At", "Length"])
        self.historyTable.horizontalHeader().setSectionResizeMode(0, QHeaderView.Stretch)
        self.historyTable.horizontalHeader().setSectionResizeMode(1, QHeaderView.ResizeToContents)
        self.historyTable.horizontalHeader().setSectionResizeMode(2, QHeaderView.ResizeToContents)
        self.historyTable.verticalHeader().setVisible(False)
        self.historyTable.setSelectionBehavior(QAbstractItemView.SelectRows)
        self.historyTable.setEditTriggers(QTableWidget.NoEditTriggers)
        self.historyTable.setShowGrid(False)
        layout.addWidget(self.historyTable)

        btnLayout = QHBoxLayout()
        self.btnCopyHistory = QPushButton("Copy Selected")
        self.btnCopyHistory.setFont(QFont("Segoe UI", 14, QFont.Bold))
        self.btnCopyHistory.clicked.connect(self.copy_selected_history)
        btnLayout.addWidget(self.btnCopyHistory)

        self.btnDeleteHistory = QPushButton("Delete Selected")
        self.btnDeleteHistory.setFont(QFont("Segoe UI", 14, QFont.Bold))
        self.btnDeleteHistory.clicked.connect(self.delete_selected_history)
        btnLayout.addWidget(self.btnDeleteHistory)

        btnLayout.addStretch()
        layout.addLayout(btnLayout)

    def _setup_preset_color_tab(self):
        layout = QVBoxLayout(self.tab_preset_color)
        layout.setSpacing(20)

        self.presetCombo = QComboBox()
        self.presetCombo.addItems(["Default", "Dark", "Colorful", "Solarized", "Pastel"])
        self.presetCombo.currentIndexChanged.connect(self.apply_preset)
        layout.addWidget(self.presetCombo)

    def apply_preset(self):
        preset = self.presetCombo.currentText()
        if preset == "Default":
            self.current_style = {
                "background": "#ffffff",
                "text": "#6b7280",
                "header": "#111827",
                "subheader": "#4b5563",
                "button_bg": "#111827",
                "button_text": "white",
                "input_bg": "#f9fafb",
                "input_text": "#111827",
                "table_bg": "#f9fafb",
                "table_header": "#e5e7eb",
                "table_item": "#374151",
                "selected_bg": "#2563eb",
                "selected_text": "white",
                "tab_bg": "#f3f4f6",
                "tab_text": "#6b7280",
                "tab_selected_bg": "#111827",
                "tab_selected_text": "white",
                "checkbox_text": "#374151"
            }
        elif preset == "Dark":
            self.current_style = {
                "background": "#121212",
                "text": "#ffffff",
                "header": "#ffffff",
                "subheader": "#cccccc",
                "button_bg": "#1f1f1f",
                "button_text": "white",
                "input_bg": "#1e1e1e",
                "input_text": "#ffffff",
                "table_bg": "#1e1e1e",
                "table_header": "#333333",
                "table_item": "#ffffff",
                "selected_bg": "#2563eb",
                "selected_text": "white",
                "tab_bg": "#000000",
                "tab_text": "#ffffff",
                "tab_selected_bg": "#111827",
                "tab_selected_text": "white",
                "checkbox_text": "#ffffff"
            }
        elif preset == "Colorful":
            self.current_style = {
                "background": "#ffeb3b",
                "text": "#000000",
                "header": "#f44336",
                "subheader": "#3f51b5",
                "button_bg": "#4caf50",
                "button_text": "white",
                "input_bg": "#ffffff",
                "input_text": "#000000",
                "table_bg": "#ffffff",
                "table_header": "#e0e0e0",
                "table_item": "#000000",
                "selected_bg": "#4caf50",
                "selected_text": "white",
                "tab_bg": "#ffeb3b",
                "tab_text": "#000000",
                "tab_selected_bg": "#4caf50",
                "tab_selected_text": "white",
                "checkbox_text": "#374151"
            }
        elif preset == "Solarized":
            self.current_style = {
                "background": "#f0e6d2",
                "text": "#657b83",
                "header": "#586e75",
                "subheader": "#93a1a1",
                "button_bg": "#268bd2",
                "button_text": "white",
                "input_bg": "#d6cabd",
                "input_text": "#657b83",
                "table_bg": "#d6cabd",
                "table_header": "#93a1a1",
                "table_item": "#657b83",
                "selected_bg": "#268bd2",
                "selected_text": "white",
                "tab_bg": "#d6cabd",
                "tab_text": "#657b83",
                "tab_selected_bg": "#268bd2",
                "tab_selected_text": "white",
                "checkbox_text": "#586e75"
            }
        elif preset == "Pastel":
            self.current_style = {
                "background": "#ffe6f0",
                "text": "#5e5e5e",
                "header": "#423737",
                "subheader": "#6a6a6a",
                "button_bg": "#a1cfff",
                "button_text": "white",
                "input_bg": "#fdf9f9",
                "input_text": "#5e5e5e",
                "table_bg": "#fdf9f9",
                "table_header": "#dedede",
                "table_item": "#5e5e5e",
                "selected_bg": "#a1cfff",
                "selected_text": "white",
                "tab_bg": "#fdf9f9",
                "tab_text": "#5e5e5e",
                "tab_selected_bg": "#a1cfff",
                "tab_selected_text": "white",
                "checkbox_text": "#6a6a6a"
            }
        self.apply_styles()

if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = PasswordGeneratorApp()
    window.show()
    sys.exit(app.exec_())

