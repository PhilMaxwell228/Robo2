python install - https://www.python.org/downloads/

vc code - https://code.visualstudio.com/

расширение питона

новый файл main.py


PS:
Get-ExecutionPolicy

Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

Y

python -m venv .venv   # Или python3 -m venv .venv - создание виртуального окружения

.venv\Scripts\Activate.ps1 - ps
.\venv\Scripts\activate.bat - cmd

пакеты:

pip install PyQt5
pip show PyQt5

мб это
pip install PyQt5 opencv-python numpy
если несколько версий питона
python3 -m pip install PyQt5 opencv-python numpy

проверка версии питона
python --version
pip --version

 мб в дебагинг меню потыкать состандартным интерпритатором







```
from PyQt5.QtWidgets import *
from PyQt5.QtCore import *
from PyQt5.QtGui import *

import sys
import datetime
import os

class ARM95_GUI(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("ARM95 - Управление роботизированной ячейкой")
        self.setGeometry(100, 100, 1200, 800)
        self.setStyleSheet("font-size: 14px;")

        # Инициализация логирования
        self.log_file_path = "arm95_log.txt"
        self.log_file = open(self.log_file_path, "a")

        # Инициализация UI
        self.init_ui()

    def init_ui(self):
        main_widget = QWidget()
        layout = QVBoxLayout()

        # 1. Кнопки управления
        control_buttons = QHBoxLayout()
        self.power_button = QPushButton("Выкл")
        self.pause_button = QPushButton("Пауза")
        self.emergency_button = QPushButton("Экстренное торможение")
        control_buttons.addWidget(self.power_button)
        control_buttons.addWidget(self.pause_button)
        control_buttons.addWidget(self.emergency_button)

        # 2. Джойстик (Move L / Move J)
        joystick_group = QGroupBox("Ручное управление")
        joystick_layout = QVBoxLayout()
        self.move_mode = QComboBox()
        self.move_mode.addItems(["Move L", "Move J"])
        self.move_mode.currentIndexChanged.connect(self.change_move_mode)

        joystick_controls = QHBoxLayout()
        self.joystick_axes = [QPushButton("X+"), QPushButton("X-"),
                              QPushButton("Y+"), QPushButton("Y-"),
                              QPushButton("Z+"), QPushButton("Z-")]
        for btn in self.joystick_axes:
            joystick_controls.addWidget(btn)

        joystick_layout.addWidget(self.move_mode)
        joystick_layout.addLayout(joystick_controls)
        joystick_group.setLayout(joystick_layout)

        # 3. Схват
        gripper_group = QGroupBox("Схват")
        gripper_layout = QHBoxLayout()
        self.gripper_state = QLabel("Состояние: 0")
        gripper_button = QPushButton("Изменить состояние")
        gripper_layout.addWidget(self.gripper_state)
        gripper_layout.addWidget(gripper_button)
        gripper_group.setLayout(gripper_layout)

        # 4. Состояние моторов
        motor_group = QGroupBox("Состояние моторов")
        self.motor_labels = []
        motor_layout = QVBoxLayout()
        for i in range(6):
            lbl = QLabel(f"Мотор {i+1}: Температура: 0°C, Положение: 0°")
            self.motor_labels.append(lbl)
            motor_layout.addWidget(lbl)
        motor_group.setLayout(motor_layout)

        # 5. Текущие координаты и статус
        status_group = QGroupBox("Текущее состояние")
        status_layout = QVBoxLayout()
        self.tool_pose = QLabel("Текущая позиция: [0, 0, 0, 0, 0, 0]")
        self.robot_status = QLabel("Статус: Ожидание")
        self.traffic_light = QLabel("Светофор: 🔵")
        status_layout.addWidget(self.tool_pose)
        status_layout.addWidget(self.robot_status)
        status_layout.addWidget(self.traffic_light)
        status_group.setLayout(status_layout)

        # 6. Логирование
        log_group = QGroupBox("Лог системы")
        self.log_text = QTextEdit()
        self.log_text.setReadOnly(True)
        self.save_log_button = QPushButton("Сохранить лог")
        log_layout = QVBoxLayout()
        log_layout.addWidget(self.log_text)
        log_layout.addWidget(self.save_log_button)
        log_group.setLayout(log_layout)

        # 7. Программирование
        program_group = QGroupBox("Автоматическое управление")
        program_layout = QVBoxLayout()
        self.program_coords = QTextEdit()
        self.program_coords.setPlaceholderText("Координаты: [[x,y,z,...], ...]")
        self.loop_checkbox = QCheckBox("Зациклить выполнение")
        self.loop_count = QSpinBox()
        self.loop_count.setValue(1)
        self.load_program_button = QPushButton("Загрузить программу")
        program_layout.addWidget(self.program_coords)
        program_layout.addWidget(self.loop_checkbox)
        program_layout.addWidget(self.loop_count)
        program_layout.addWidget(self.load_program_button)
        program_group.setLayout(program_layout)

        # 8. Видео
        video_group = QGroupBox("Видео с камеры")
        self.video_label = QLabel("Видео не подключено")
        self.video_label.setAlignment(Qt.AlignCenter)
        video_layout = QVBoxLayout()
        video_layout.addWidget(self.video_label)
        video_group.setLayout(video_layout)

        # 9. Распознанные объекты
        detection_group = QGroupBox("Распознанные объекты")
        self.detection_label = QLabel("Нет объектов")
        detection_layout = QVBoxLayout()
        detection_layout.addWidget(self.detection_label)
        detection_group.setLayout(detection_layout)

        # Добавляем все элементы
        layout.addLayout(control_buttons)
        layout.addWidget(joystick_group)
        layout.addWidget(gripper_group)
        layout.addWidget(motor_group)
        layout.addWidget(status_group)
        layout.addWidget(program_group)
        layout.addWidget(video_group)
        layout.addWidget(detection_group)
        layout.addWidget(log_group)

        main_widget.setLayout(layout)
        self.setCentralWidget(main_widget)

        # События
        self.power_button.clicked.connect(self.power_off)
        self.pause_button.clicked.connect(self.pause)
        self.emergency_button.clicked.connect(self.emergency_stop)
        gripper_button.clicked.connect(self.toggle_gripper)
        self.save_log_button.clicked.connect(self.save_log)
        self.load_program_button.clicked.connect(self.load_program)

        # Тестовое обновление
        self.test_update()

    # --- Функции ---
    def change_move_mode(self):
        mode = self.move_mode.currentText()
        self.log(f"Робот переключен в режим: {mode}")

    def power_off(self):
        self.log("Робот выключен")
        self.robot_status.setText("Статус: Выключен")
        self.traffic_light.setText("Светофор: 🔵")

    def pause(self):
        self.log("Робот приостановлен")
        self.robot_status.setText("Статус: Пауза")
        self.traffic_light.setText("Светофор: 🟡")

    def emergency_stop(self):
        self.log("Аварийная остановка")
        self.robot_status.setText("Статус: Авария")
        self.traffic_light.setText("Светофор: 🔴")

    def toggle_gripper(self):
        current = self.gripper_state.text().split(":")[1].strip()
        new_state = "1" if current == "0" else "0"
        self.gripper_state.setText(f"Состояние: {new_state}")
        self.log(f"Схват изменен: {new_state}")

    def update_motor_status(self):
        for i in range(6):
            self.motor_labels[i].setText(f"Мотор {i+1}: Температура: 35°C, Положение: 45°")

    def test_update(self):
        self.update_motor_status()
        self.tool_pose.setText("Текущая позиция: [100, 200, 300, 0, 0, 0]")
        self.robot_status.setText("Статус: Работа")
        self.traffic_light.setText("Светофор: 🟢")

    def log(self, message):
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        log_entry = f"[{timestamp}] {message}\n"
        self.log_text.append(log_entry)
        self.log_file.write(log_entry)
        self.log_file.flush()

    def save_log(self):
        path = QFileDialog.getSaveFileName(self, "Сохранить лог", "", "Текстовые файлы (*.txt)")
        if path[0]:
            with open(path[0], 'w') as f:
                f.write(self.log_text.toPlainText())
            self.log("Лог сохранен")

    def load_program(self):
        path, _ = QFileDialog.getOpenFileName(self, "Загрузить программу", "", "Текстовые файлы (*.txt)")
        if path:
            try:
                with open(path, 'r') as f:
                    content = f.read()
                self.program_coords.setText(content)
                self.log("Программа загружена")
            except Exception as e:
                self.log(f"Ошибка загрузки программы: {e}")

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = ARM95_GUI()
    window.show()
    sys.exit(app.exec_())
