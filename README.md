# =============================
# Student Management System
# Tkinter + MySQL CRUD Application
# =============================

# Before running:
# 1. Install MySQL
# 2. Create database:
#
#    CREATE DATABASE student_management;
#
# 3. Install mysql connector:
#    pip install mysql-connector-python
#
# 4. Update MySQL username/password below

import tkinter as tk
from tkinter import ttk, messagebox
import mysql.connector

# =============================
# MYSQL CONNECTION
# =============================

db = mysql.connector.connect(
    host="localhost",
    user="root",          # change your mysql username
    password="1234",      # change your mysql password
    database="student_management"
)

cursor = db.cursor()

# =============================
# CREATE TABLE
# =============================

cursor.execute("""
CREATE TABLE IF NOT EXISTS students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    age INT,
    course VARCHAR(100),
    email VARCHAR(100)
)
""")

# =============================
# MAIN WINDOW
# =============================

root = tk.Tk()
root.title("Student Management System")
root.geometry("900x600")
root.configure(bg="#f0f0f0")

# =============================
# VARIABLES
# =============================

name_var = tk.StringVar()
age_var = tk.StringVar()
course_var = tk.StringVar()
email_var = tk.StringVar()

# =============================
# FUNCTIONS
# =============================

def clear_fields():
    name_var.set("")
    age_var.set("")
    course_var.set("")
    email_var.set("")


def add_student():
    name = name_var.get()
    age = age_var.get()
    course = course_var.get()
    email = email_var.get()

    if name == "" or age == "" or course == "" or email == "":
        messagebox.showerror("Error", "All fields are required")
        return

    sql = "INSERT INTO students (name, age, course, email) VALUES (%s, %s, %s, %s)"
    values = (name, age, course, email)

    cursor.execute(sql, values)
    db.commit()

    messagebox.showinfo("Success", "Student added successfully")

    clear_fields()
    show_students()


def show_students():
    tree.delete(*tree.get_children())

    cursor.execute("SELECT * FROM students")
    rows = cursor.fetchall()

    for row in rows:
        tree.insert("", tk.END, values=row)


def delete_student():
    selected = tree.focus()

    if not selected:
        messagebox.showerror("Error", "Select a student")
        return

    data = tree.item(selected)
    student_id = data['values'][0]

    cursor.execute("DELETE FROM students WHERE id=%s", (student_id,))
    db.commit()

    messagebox.showinfo("Deleted", "Student deleted")

    show_students()


def get_selected(event):
    selected = tree.focus()

    if not selected:
        return

    data = tree.item(selected)
    row = data['values']

    name_var.set(row[1])
    age_var.set(row[2])
    course_var.set(row[3])
    email_var.set(row[4])


def update_student():
    selected = tree.focus()

    if not selected:
        messagebox.showerror("Error", "Select a student")
        return

    data = tree.item(selected)
    student_id = data['values'][0]

    sql = """
    UPDATE students
    SET name=%s, age=%s, course=%s, email=%s
    WHERE id=%s
    """

    values = (
        name_var.get(),
        age_var.get(),
        course_var.get(),
        email_var.get(),
        student_id
    )

    cursor.execute(sql, values)
    db.commit()

    messagebox.showinfo("Updated", "Student updated")

    clear_fields()
    show_students()


# =============================
# TITLE
# =============================

title = tk.Label(
    root,
    text="Student Management System",
    font=("Arial", 22, "bold"),
    bg="#4a7abc",
    fg="white",
    pady=10
)

title.pack(fill=tk.X)

# =============================
# FORM FRAME
# =============================

form_frame = tk.Frame(root, bg="white", padx=20, pady=20)
form_frame.pack(fill=tk.X, padx=20, pady=10)

# Name
tk.Label(form_frame, text="Name", font=("Arial", 12), bg="white").grid(row=0, column=0, padx=10, pady=10)

tk.Entry(form_frame, textvariable=name_var, font=("Arial", 12), width=30).grid(row=0, column=1)

# Age
tk.Label(form_frame, text="Age", font=("Arial", 12), bg="white").grid(row=0, column=2, padx=10)

tk.Entry(form_frame, textvariable=age_var, font=("Arial", 12), width=30).grid(row=0, column=3)

# Course
tk.Label(form_frame, text="Course", font=("Arial", 12), bg="white").grid(row=1, column=0, padx=10, pady=10)

tk.Entry(form_frame, textvariable=course_var, font=("Arial", 12), width=30).grid(row=1, column=1)

# Email
tk.Label(form_frame, text="Email", font=("Arial", 12), bg="white").grid(row=1, column=2, padx=10)

tk.Entry(form_frame, textvariable=email_var, font=("Arial", 12), width=30).grid(row=1, column=3)

# =============================
# BUTTONS
# =============================

button_frame = tk.Frame(root, bg="#f0f0f0")
button_frame.pack(pady=10)

btn_style = {
    "font": ("Arial", 12, "bold"),
    "width": 15,
    "bd": 0,
    "fg": "white",
    "pady": 8
}

tk.Button(
    button_frame,
    text="Add Student",
    bg="#28a745",
    command=add_student,
    **btn_style
).grid(row=0, column=0, padx=10)

tk.Button(
    button_frame,
    text="Update Student",
    bg="#007bff",
    command=update_student,
    **btn_style
).grid(row=0, column=1, padx=10)

tk.Button(
    button_frame,
    text="Delete Student",
    bg="#dc3545",
    command=delete_student,
    **btn_style
).grid(row=0, column=2, padx=10)

tk.Button(
    button_frame,
    text="Clear",
    bg="#6c757d",
    command=clear_fields,
    **btn_style
).grid(row=0, column=3, padx=10)

# =============================
# TABLE FRAME
# =============================

table_frame = tk.Frame(root)
table_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=10)

scroll_y = tk.Scrollbar(table_frame, orient=tk.VERTICAL)

tree = ttk.Treeview(
    table_frame,
    columns=("ID", "Name", "Age", "Course", "Email"),
    show="headings",
    yscrollcommand=scroll_y.set
)

scroll_y.config(command=tree.yview)
scroll_y.pack(side=tk.RIGHT, fill=tk.Y)

# Headings
tree.heading("ID", text="ID")
tree.heading("Name", text="Name")
tree.heading("Age", text="Age")
tree.heading("Course", text="Course")
tree.heading("Email", text="Email")

# Column Width
tree.column("ID", width=50)
tree.column("Name", width=180)
tree.column("Age", width=80)
tree.column("Course", width=150)
tree.column("Email", width=250)

tree.pack(fill=tk.BOTH, expand=True)

# =============================
# BIND SELECT EVENT
# =============================

tree.bind("<ButtonRelease-1>", get_selected)

# =============================
# LOAD DATA
# =============================

show_students()

# =============================
# RUN APPLICATION
# =============================

root.mainloop()
```

## Steps to Run

### 1. Create Database in MySQL

```sql
CREATE DATABASE student_management;
```

---

### 2. Install Required Package

```bash
pip install mysql-connector-python
```

---

### 3. Save File

Save as:

```bash
student_management.py
```

---

### 4. Run

```bash
python student_management.py
```

This system includes:

* Add Student
* View Students
* Update Student
* Delete Student
* MySQL Database Storage
* Tkinter GUI Table View
* Auto ID generation
* Form Clear Button
