# oliwia_znowu_nie_byla_w_szkole
import sys
import os
from PyQt5 import QtGui, QtCore
from PyQt5.QtGui import QIcon
from PyQt5.QtWidgets import QApplication, QMainWindow, QListWidget, QListWidgetItem

class TestListView(QListWidget):
    def __init__(self, parent=None):
        super(TestListView, self).__init__(parent)
        self.setAcceptDrops(True)
        self.setIconSize(QtCore.QSize(72, 72))
        
    def dragEnterEvent(self, e):
        if e.mimeData().hasUrls():
            e.accept()
        else:
            e.ignore()

    def dragMoveEvent(self, e):
        if e.mimeData().hasUrls():
            e.setDropAction(QtCore.Qt.CopyAction)
            e.accept()
        else:
            e.ignore()

    def dropEvent(self, e):
        if e.mimeData().hasUrls():
            e.setDropAction(QtCore.Qt.CopyAction)
            e.accept()
            links = []
            for url in e.mimeData().urls():
                links.append(str(url.toLocalFile()))
                print(str(url.toLocalFile()))
            self.pictureDropped(links)
        else:
            e.ignore()
    
    def pictureDropped(self, links):
        for url in links:
            if os.path.exists(url):
                print(url)
                icon = QIcon(url)
                item = QListWidgetItem(url, self)
                item.setIcon(icon)
                item.setStatusTip(url)

class MainForm(QMainWindow):
    def __init__(self, parent=None):
        super(MainForm, self).__init__(parent)
        self.view = TestListView(self)
        self.setCentralWidget(self.view)
        self.resize(480, 320)

def main():
    app = QApplication(sys.argv)
    form = MainForm()
    form.show()
    sys.exit(app.exec_())

if __name__ == '__main__':
    main()

