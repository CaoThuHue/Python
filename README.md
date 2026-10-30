
import tkinter as tk
from tkinter import messagebox
import xml.etree.ElementTree as ET
#import json
import math

BG = '#FCE7F3'
CARD = '#FFFFFF'
PINK = '#BE185D'
TEXT = '#831843'

class ProductManager:
    def __init__(self):
        self.filename = 'products.xml'
        #self.filename = 'products.json'
        self.products = []
        self.load_products()
##########
def save_products(self):
    try:
        with open(self.filename, 'w', encoding='utf-8') as f:
            json.dump(self.products, f, ensure_ascii=False, indent=4)
        return True
    except (OSError, ValueError) as e:
        messagebox.showerror('Lỗi lưu JSON', str(e))
        return False

def load_products(self):
    try:
        with open(self.filename, 'r', encoding='utf-8') as f:
            self.products = json.load(f)
    except FileNotFoundError:
        self.products = []
    except json.JSONDecodeError:
        messagebox.showerror('Lỗi', 'File JSON không hợp lệ')
        self.products = []
##########
    def save_products(self):
        try:
            root_xml = ET.Element('products')

            for p in self.products:
                sp = ET.SubElement(root_xml, 'product')
                ET.SubElement(sp, 'name').text = p['name']
                ET.SubElement(sp, 'price').text = str(p['price'])

            tree = ET.ElementTree(root_xml)
            ET.indent(tree, space='    ')
            tree.write(
                self.filename,
                encoding='utf-8',
                xml_declaration=True
            )
            return True

        except (OSError, ValueError, TypeError) as e:
            messagebox.showerror('Lỗi lưu XML', str(e))
            return False

    def load_products(self):
        try:
            tree = ET.parse(self.filename)
            root_xml = tree.getroot()

            if root_xml.tag != 'products':
                raise ValueError('Thẻ gốc phải là products')

            data = []

            for sp in root_xml.findall('product'):
                name = sp.findtext('name')
                price = float(sp.findtext('price'))

                if not name or not name.strip():
                    raise ValueError('Tên không hợp lệ')

                if not math.isfinite(price) or price <= 0:
                    raise ValueError('Giá không hợp lệ')

                data.append({
                    'name': name,
                    'price': price
                })

            self.products = data

        except FileNotFoundError:
            self.products = []

        except (ET.ParseError, ValueError, TypeError, OSError) as e:
            messagebox.showerror('Lỗi đọc XML', str(e))
            self.products = []

    def add_product(self, product):
        self.products.append(product)

        if not self.save_products():
            self.products.pop()
            return False

        return True

    def update_product(self, index, product):
        if not 0 <= index < len(self.products):
            return False

        old = self.products[index]
        self.products[index] = product

        if not self.save_products():
            self.products[index] = old
            return False

        return True

    def delete_product(self, index):
        if not 0 <= index < len(self.products):
            return False

        old = self.products.pop(index)

        if not self.save_products():
            self.products.insert(index, old)
            return False

        return True

class ProductApp:
    def __init__(self, root):
        self.root = root
        self.manager = ProductManager()
        self.selected_index = None

        self.root.title('Quản lý sản phẩm')
        self.root.geometry('650x590')
        self.root.configure(bg=BG)
        self.root.columnconfigure(0, weight=1)
        self.root.rowconfigure(2, weight=1)

        tk.Label(self.root,text='QUẢN LÝ SẢN PHẨM',bg=BG,fg=PINK,font=('Arial', 19, 'bold')).grid(row=0, column=0,padx=18, pady=(18, 10))

        form = tk.Frame(self.root,bg=CARD,padx=20,pady=17,highlightbackground='#F9A8D4',highlightthickness=1)
        form.grid(row=1, column=0,sticky='ew', padx=18, pady=10)
        for i in range(4):
            form.columnconfigure(i, weight=1)
        tk.Label(form,text='THÔNG TIN SẢN PHẨM',bg=CARD,fg=PINK,font=('Arial', 13, 'bold')).grid(row=0, column=0,columnspan=4, sticky='w',pady=(0, 12))
        tk.Label(form, text='Tên sản phẩm',bg=CARD, fg=TEXT).grid(row=1, column=0, sticky='w', pady=8)
        self.entry_name = tk.Entry(form,bg='#FFF7FB',font=('Arial', 11),relief='solid',bd=1)
        self.entry_name.grid(row=1, column=1,columnspan=3, sticky='ew',padx=(15, 0), pady=8, ipady=6)
        tk.Label(form, text='Giá sản phẩm',bg=CARD, fg=TEXT).grid(row=2, column=0, sticky='w', pady=8)
        self.entry_price = tk.Entry(form,bg='#FFF7FB',font=('Arial', 11),relief='solid',bd=1)
        self.entry_price.grid(row=2, column=1,columnspan=3, sticky='ew',padx=(15, 0), pady=8, ipady=6)
        self.btn_them = tk.Button(form,text='Thêm',command=self.themsp,bg='#16A34A',fg='white',width=12,font=('Arial', 10, 'bold'))
        self.btn_them.grid(row=3, column=0, padx=10, pady=10)

        tk.Button(form,text='Sửa',command=self.suasp,bg='#2563EB',fg='white',width=12,font=('Arial', 10, 'bold')).grid(row=3, column=1, padx=10, pady=10)
        tk.Button(form,text='Xóa',command=self.xoasp,bg='#DC2626',fg='white',width=12,font=('Arial', 10, 'bold')).grid(row=3, column=2, padx=10, pady=10)
        tk.Button(form,text='Làm mới',command=self.lammoi,bg='#64748B',fg='white',width=12,font=('Arial', 10, 'bold')).grid(row=3, column=3, padx=10, pady=10)

        list_frame = tk.Frame(self.root,bg=CARD,padx=18,pady=15,highlightbackground='#F9A8D4',highlightthickness=1)
        list_frame.grid(row=2, column=0,sticky='nsew', padx=18, pady=10)
        list_frame.columnconfigure(0, weight=1)
        list_frame.rowconfigure(1, weight=1)
        tk.Label(list_frame,text='DANH SÁCH SẢN PHẨM',bg=CARD,fg=PINK,font=('Arial', 13, 'bold')).grid(row=0, column=0, sticky='w', pady=(0, 10))
        self.count_label = tk.Label(list_frame,text='Tổng số: 0 sản phẩm',bg=CARD,fg=TEXT)
        self.count_label.grid(row=0, column=1, pady=(0, 10))
        self.listbox = tk.Listbox(list_frame,font=('Arial', 11),bg='#FFF7FB',selectbackground='#F9A8D4',selectforeground=TEXT,activestyle='none',exportselection=False,height=10)
        self.listbox.grid(row=1, column=0,columnspan=2, sticky='nsew')
        scrollbar = tk.Scrollbar(list_frame,command=self.listbox.yview)
        scrollbar.grid(row=1, column=2, sticky='ns')
        self.listbox.config(yscrollcommand=scrollbar.set)
        self.listbox.bind('<<ListboxSelect>>', self.chonsp)
        self.hienthi()

    def hienthi(self):
        self.listbox.delete(0, tk.END)
        for p in self.manager.products:
            self.listbox.insert(tk.END,f"{p['name']} - {p['price']:g} VND")
        self.count_label.config(text=f'Tổng số: {len(self.manager.products)} sản phẩm')

    def laysp(self):
        name = self.entry_name.get().strip()
        price_text = self.entry_price.get().strip()
        if not name:
            messagebox.showwarning(
                'Cảnh báo',
                'Tên sản phẩm không được để trống'
            )
            return None
        try:
            price = float(price_text)
        except ValueError:
            messagebox.showerror(
                'Lỗi', 'Giá sản phẩm phải là số'
            )
            return None
        if not math.isfinite(price) or price <= 0:
            messagebox.showerror(
                'Lỗi', 'Giá phải lớn hơn 0 và hữu hạn'
            )
            return None
        return {'name': name, 'price': price}
    def lammoi(self):
        self.selected_index = None
        self.listbox.selection_clear(0, tk.END)
        self.entry_name.delete(0, tk.END)
        self.entry_price.delete(0, tk.END)
        self.btn_them.config(
            text='Thêm',
            command=self.themsp,
            bg='#16A34A'
        )
        self.entry_name.focus_set()
    def chonsp(self, event):
        selected = self.listbox.curselection()
        if not selected:
            return
        index = selected[0]
        if index >= len(self.manager.products):
            return
        self.selected_index = index
        product = self.manager.products[index]
        self.entry_name.delete(0, tk.END)
        self.entry_price.delete(0, tk.END)
        self.entry_name.insert(0, product['name'])
        self.entry_price.insert(0, f"{product['price']:g}")
        self.btn_them.config(
            text='Cập nhật',
            command=self.suasp,
            bg='#2563EB'
        )
    def themsp(self):
        product = self.laysp()
        if product is None:
            return
        if not self.manager.add_product(product):
            return
        self.lammoi()
        self.hienthi()
        messagebox.showinfo('Thành công', 'Đã thêm và lưu sản phẩm')

    def suasp(self):
        if self.selected_index is None:
            messagebox.showwarning('Cảnh báo', 'Hãy chọn sản phẩm cần sửa')
            return
        product = self.laysp()
        if product is None:
            return
        if not self.manager.update_product(self.selected_index, product):
            return

        self.lammoi()
        self.hienthi()
        messagebox.showinfo('Thành công', 'Đã cập nhật sản phẩm')

    def xoasp(self):
        if self.selected_index is None:
            messagebox.showwarning('Cảnh báo', 'Hãy chọn sản phẩm cần xóa')
            return

        if not messagebox.askyesno('Xác nhận', 'Bạn có chắc muốn xóa?'):
            return

        if not self.manager.delete_product(self.selected_index):
            return

        self.lammoi()
        self.hienthi()
        messagebox.showinfo('Thành công', 'Đã xóa sản phẩm')

# CHAY UNG DUNG
if __name__ == '__main__':
    root = tk.Tk()
    app = ProductApp(root)
    root.mainloop()
