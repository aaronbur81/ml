import sqlite3
from PyQt6.QtWidgets import QApplication, QMainWindow, QTabWidget, QWidget, QVBoxLayout, QTextEdit, QPushButton, QListWidget, QMessageBox, QInputDialog

class ContractScopeApp(QMainWindow):
    def __init__(self):
        super().__init__()

        self.setWindowTitle("Contract Scope Tracker")
        self.setGeometry(100, 100, 800, 600)

        self.conn = sqlite3.connect("contracts.db")
        self.create_tables()

        self.tabs = QTabWidget()
        self.setCentralWidget(self.tabs)

        self.noresco_tab = self.create_contract_tab("NORESCO Proposed", ["PA", "IGA", "100% Design"])
        self.owner_tab = self.create_contract_tab("Owner Proposed", ["PA Task Order", "IGA Task Order", "Construction Task Order", "100% Design Comments"])
        self.subcontracts_tab = self.create_contract_tab("Subcontracts", ["IGA Bid Documents", "100% Review Comments"])

        self.tabs.addTab(self.noresco_tab, "NORESCO Proposed")
        self.tabs.addTab(self.owner_tab, "Owner Proposed")
        self.tabs.addTab(self.subcontracts_tab, "Subcontracts")

    def create_tables(self):
        """ Create database tables if they don't exist """
        with self.conn:
            self.conn.execute('''
                CREATE TABLE IF NOT EXISTS contracts (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    contract_type TEXT NOT NULL,
                    ecm_name TEXT NOT NULL
                )
            ''')
            self.conn.execute('''
                CREATE TABLE IF NOT EXISTS scopes (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    contract_id INTEGER NOT NULL,
                    scope_text TEXT,
                    FOREIGN KEY (contract_id) REFERENCES contracts(id)
                )
            ''')

    def create_contract_tab(self, contract_type, ecm_list):
        """ Create a tab with contract scope functionality """
        tab = QWidget()
        layout = QVBoxLayout()

        self.list_widget = QListWidget()
        for ecm in ecm_list:
            self.list_widget.addItem(ecm)
        layout.addWidget(self.list_widget)

        self.scope_text = QTextEdit()
        layout.addWidget(self.scope_text)

        btn_add = QPushButton("Add Scope Item")
        btn_edit = QPushButton("Edit Selected")
        btn_delete = QPushButton("Delete Selected")
        btn_check = QPushButton("Check Coverage")

        layout.addWidget(btn_add)
        layout.addWidget(btn_edit)
        layout.addWidget(btn_delete)
        layout.addWidget(btn_check)

        btn_add.clicked.connect(lambda: self.add_scope(contract_type))
        btn_edit.clicked.connect(self.edit_scope)
        btn_delete.clicked.connect(self.delete_scope)

        tab.setLayout(layout)
        return tab

    def add_scope(self, contract_type):
        """ Add a new scope entry to the database """
        ecm_name, ok = QInputDialog.getText(self, "Add ECM", "Enter ECM Name:")
        if not ok or not ecm_name.strip():
            return

        scope_text, ok = QInputDialog.getMultiLineText(self, "Add Scope", "Enter Scope Details:")
        if not ok or not scope_text.strip():
            return

        with self.conn:
            cur = self.conn.cursor()
            cur.execute("INSERT INTO contracts (contract_type, ecm_name) VALUES (?, ?)", (contract_type, ecm_name))
            contract_id = cur.lastrowid
            cur.execute("INSERT INTO scopes (contract_id, scope_text) VALUES (?, ?)", (contract_id, scope_text))
            self.conn.commit()

        self.list_widget.addItem(ecm_name)

    def edit_scope(self):
        """ Edit an existing scope entry """
        selected_item = self.list_widget.currentItem()
        if not selected_item:
            QMessageBox.warning(self, "Edit Scope", "No ECM selected.")
            return

        new_scope_text, ok = QInputDialog.getMultiLineText(self, "Edit Scope", "Modify Scope Details:", self.scope_text.toPlainText())
        if ok:
            ecm_name = selected_item.text()
            with self.conn:
                cur = self.conn.cursor()
                cur.execute("UPDATE scopes SET scope_text = ? WHERE contract_id = (SELECT id FROM contracts WHERE ecm_name = ?)", (new_scope_text, ecm_name))
                self.conn.commit()
            self.scope_text.setPlainText(new_scope_text)

    def delete_scope(self):
        """ Delete an ECM and its associated scope """
        selected_item = self.list_widget.currentItem()
        if not selected_item:
            QMessageBox.warning(self, "Delete Scope", "No ECM selected.")
            return

        ecm_name = selected_item.text()
        reply = QMessageBox.question(self, "Delete Scope", f"Are you sure you want to delete '{ecm_name}'?", QMessageBox.StandardButton.Yes | QMessageBox.StandardButton.No)
        if reply == QMessageBox.StandardButton.Yes:
            with self.conn:
                cur = self.conn.cursor()
                cur.execute("DELETE FROM contracts WHERE ecm_name = ?", (ecm_name,))
                cur.execute("DELETE FROM scopes WHERE contract_id NOT IN (SELECT id FROM contracts)")  # Clean orphaned scopes
                self.conn.commit()
            self.list_widget.takeItem(self.list_widget.row(selected_item))
            self.scope_text.clear()

if __name__ == "__main__":
    app = QApplication([])
    window = ContractScopeApp()
    window.show()
    app.exec()