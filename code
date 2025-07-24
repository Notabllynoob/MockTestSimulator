import sys
import re
from PyQt5.QtWidgets import (QApplication, QMainWindow, QWidget, QLabel, QVBoxLayout,
                             QHBoxLayout, QRadioButton, QButtonGroup, QPushButton,
                             QFileDialog, QMessageBox, QInputDialog, QStackedWidget)
from PyQt5.QtCore import QTimer, Qt


class TestSlide(QWidget):
    def __init__(self, question_text, options):
        super().__init__()
        self.layout = QVBoxLayout(self)
        self.question_label = QLabel(question_text)
        self.question_label.setStyleSheet("font-size: 16px; font-weight: bold;")
        self.question_label.setWordWrap(True)
        self.layout.addWidget(self.question_label)

        self.btn_group = QButtonGroup(self)
        for opt in options:
            radio = QRadioButton(opt)
            radio.setStyleSheet("font-size: 14px;")
            self.layout.addWidget(radio)
            self.btn_group.addButton(radio)

        self.setLayout(self.layout)


class TestInterface(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Mock Test Interface - Slide View")
        self.resize(800, 600)

        self.central_widget = QWidget()
        self.setCentralWidget(self.central_widget)

        self.vlayout = QVBoxLayout(self.central_widget)

        # Timer label
        self.timer_label = QLabel("Time Left: --:--", self)
        self.timer_label.setAlignment(Qt.AlignRight | Qt.AlignTop)
        self.timer_label.setFixedHeight(50)
        self.timer_label.setStyleSheet("font-size: 16px; font-weight: bold; padding: 8px;")
        self.vlayout.addWidget(self.timer_label)

        # QStackedWidget to show one question (slide) at a time
        self.stack = QStackedWidget()
        self.vlayout.addWidget(self.stack)

        # Navigation Buttons layout
        self.nav_layout = QHBoxLayout()
        self.prev_button = QPushButton("Previous")
        self.prev_button.clicked.connect(self.go_prev)
        self.nav_layout.addWidget(self.prev_button)

        self.next_button = QPushButton("Next")
        self.next_button.clicked.connect(self.go_next)
        self.nav_layout.addWidget(self.next_button)

        self.finish_button = QPushButton("Finish")
        self.finish_button.clicked.connect(self.submit_answers)
        self.nav_layout.addWidget(self.finish_button)

        self.vlayout.addLayout(self.nav_layout)

        self.timer = QTimer(self)
        self.timer.timeout.connect(self.update_timer)
        self.time_left = 0  # in seconds

        self.slides = []
        self.questions = []

        self.load_text_and_prepare()

    def load_text_and_prepare(self):
        file_path, _ = QFileDialog.getOpenFileName(self, "Select Text File", "", "Text Files (*.txt)")
        if not file_path:
            sys.exit("No file selected, baby boy!")
        try:
            with open(file_path, "r", encoding="utf-8", errors="ignore") as f:
                file_content = f.read()
        except Exception as e:
            sys.exit("Failed to load file: " + str(e))


        question_blocks = re.findall(r"(Q[\s\.]*\d+\..*?)(?=Q[\s\.]*\d+\.|$)", file_content, re.DOTALL)

        if not question_blocks:
            QMessageBox.warning(self, "Parsing Error", "No questions found! Check your file formatting.")
            return

        self.questions = []
        for block in question_blocks:
            parts = re.split(r"\n\s*(?=[A-D]\))", block.strip())
            if len(parts) < 2:
                continue
            question_text = parts[0].strip()
            options = [p.strip() for p in parts[1:]]
            clean_options = [" ".join(opt.split()) for opt in options]
            self.questions.append((question_text, clean_options))

        if not self.questions:
            QMessageBox.warning(self, "Parsing Error", "Could not parse any valid questions!")
            return

        minutes, ok = QInputDialog.getInt(self, "Set Timer", "Enter test duration (minutes):", min=1, max=180)
        if not ok:
            sys.exit("Timer not set!")
        self.time_left = minutes * 60

        self.build_slides()
        self.timer.start(1000)

    def build_slides(self):
        self.slides = []
        for q_text, options in self.questions:
            slide = TestSlide(q_text, options)
            self.slides.append(slide)
            self.stack.addWidget(slide)

    def go_next(self):
        current_index = self.stack.currentIndex()
        if current_index < self.stack.count() - 1:
            self.stack.setCurrentIndex(current_index + 1)

    def go_prev(self):
        current_index = self.stack.currentIndex()
        if current_index > 0:
            self.stack.setCurrentIndex(current_index - 1)

    def update_timer(self):
        if self.time_left > 0:
            mins, secs = divmod(self.time_left, 60)
            self.timer_label.setText(f"Time Left: {mins:02d}:{secs:02d}")
            self.time_left -= 1
        else:
            self.timer.stop()
            self.submit_answers()

    def submit_answers(self):
        self.timer.stop()
        candidate_answers = []
        for slide in self.slides:
            checked = slide.btn_group.checkedButton()
            answer = checked.text() if checked else "Not Answered"
            candidate_answers.append(answer)

        correct_answers = []
        for i, (q_text, options) in enumerate(self.questions):
            prompt = f"Enter the correct answer for Question {i + 1} (type the exact option text):\n{q_text}\nOptions:\n" + "\n".join(options)
            ans, ok = QInputDialog.getText(self, f"Correct Answer for Q{i + 1}", prompt)
            correct_answers.append(ans.strip() if ok else "")

        right_count = 0
        wrong_count = 0
        result_details = ""
        for i, (cand, corr) in enumerate(zip(candidate_answers, correct_answers)):
            if cand.strip().lower() == corr.strip().lower():
                right_count += 1
                result_details += f"Question {i + 1}: Correct\n"
            else:
                wrong_count += 1
                result_details += f"Question {i + 1}: Wrong (Your answer: {cand}; Correct: {corr})\n"

        summary = f"Results:\nRight: {right_count}\nWrong: {wrong_count}\n\nDetails:\n{result_details}"
        QMessageBox.information(self, "Test Results", summary)


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = TestInterface()
    window.show()
    sys.exit(app.exec_())
