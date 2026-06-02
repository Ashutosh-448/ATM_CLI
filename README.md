"""
╔══════════════════════════════════════════════════════════════╗
║        NATIONAL BANK OF AKSHAY — ATM & Banking System       ║
║   PIN Login · Balance · Deposit · Withdraw · Transfer        ║
║   Mini Statement · Change PIN · Receipt · Dark UI            ║
╚══════════════════════════════════════════════════════════════╝
"""

import tkinter as tk
from tkinter import ttk, messagebox
import json, os, datetime, random, hashlib

# ── Data file ─────────────────────────────────────────────────────
DB_PATH = os.path.join(os.path.dirname(__file__), "bank_data.json")

def load_db():
    if os.path.exists(DB_PATH):
        with open(DB_PATH) as f:
            return json.load(f)
    # default accounts
    return {
        "1234567890": {
            "name":        "Akshay",
            "card_no":     "1234 5678 9012 3456",
            "pin_hash":    hashlib.sha256("1234".encode()).hexdigest(),
            "balance":     20000.0,
            "account_type":"Savings",
            "transactions":[]
        },
        "9876543210": {
            "name":        "Demo User",
            "card_no":     "9876 5432 1098 7654",
            "pin_hash":    hashlib.sha256("0000".encode()).hexdigest(),
            "balance":     5000.0,
            "account_type":"Current",
            "transactions":[]
        }
    }

def save_db(db):
    with open(DB_PATH, "w") as f:
        json.dump(db, f, indent=2)

def hash_pin(pin):
    return hashlib.sha256(pin.encode()).hexdigest()

def log_txn(db, acc_no, txn_type, amount, note=""):
    db[acc_no]["transactions"].append({
        "type":   txn_type,
        "amount": amount,
        "note":   note,
        "time":   datetime.datetime.now().strftime("%d %b %Y  %H:%M:%S"),
        "balance":db[acc_no]["balance"]
    })
    save_db(db)

# ── Palette ───────────────────────────────────────────────────────
BG      = "#0a0a14"
CARD    = "#12122a"
CARD2   = "#1a1a3e"
ACCENT  = "#1e3a5f"
BLUE    = "#1565c0"
CYAN    = "#00bcd4"
GREEN   = "#00c853"
RED     = "#f44336"
ORANGE  = "#ff9800"
GOLD    = "#ffd700"
TEXT    = "#e8eaf6"
SUBTEXT = "#9fa8da"
BORDER  = "#283593"
WHITE   = "#ffffff"

# ══════════════════════════════════════════════════════════════════
#  MAIN ATM APP
# ══════════════════════════════════════════════════════════════════
class ATMApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("🏦  National Bank of Akshay — ATM")
        self.configure(bg=BG)
        self.state("zoomed")
        self.minsize(900, 680)

        self.db         = load_db()
        self.logged_acc = None
        self._build_login()

    # ── LOGIN SCREEN ─────────────────────────────────────────────
    def _build_login(self):
        for w in self.winfo_children():
            w.destroy()

        # header
        hdr = tk.Frame(self, bg=ACCENT, pady=16)
        hdr.pack(fill="x")
        tk.Label(hdr, text="🏦  NATIONAL BANK OF AKSHAY",
                 bg=ACCENT, fg=GOLD,
                 font=("Segoe UI", 22, "bold")).pack()
        tk.Label(hdr, text="ATM & Internet Banking System",
                 bg=ACCENT, fg=SUBTEXT,
                 font=("Segoe UI", 10, "italic")).pack()

        # clock
        clock = tk.Frame(self, bg=BG, pady=6)
        clock.pack(fill="x")
        self.clock_var = tk.StringVar()
        tk.Label(clock, textvariable=self.clock_var,
                 bg=BG, fg=CYAN,
                 font=("Segoe UI", 10, "bold")).pack()
        self._tick()

        # login card
        lc = tk.Frame(self, bg=CARD,
                      highlightbackground=BORDER, highlightthickness=2)
        lc.pack(pady=60, ipadx=40, ipady=30)
        tk.Label(lc, text="🔐  ATM LOGIN", bg=CARD, fg=CYAN,
                 font=("Segoe UI", 18, "bold")).pack(pady=(0,6))
        tk.Label(lc, text="Enter your account number and PIN",
                 bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 9)).pack(pady=(0,20))

        tk.Label(lc, text="Account Number:", bg=CARD, fg=TEXT,
                 font=("Segoe UI", 10)).pack(anchor="w", padx=20)
        self.acc_entry = tk.Entry(lc, bg=CARD2, fg=TEXT,
                                  insertbackground=TEXT, relief="flat",
                                  font=("Segoe UI", 13), width=22,
                                  highlightbackground=BORDER,
                                  highlightthickness=1)
        self.acc_entry.pack(ipady=8, padx=20, pady=(2,12))

        tk.Label(lc, text="PIN:", bg=CARD, fg=TEXT,
                 font=("Segoe UI", 10)).pack(anchor="w", padx=20)
        self.pin_entry = tk.Entry(lc, bg=CARD2, fg=TEXT,
                                  insertbackground=TEXT, relief="flat",
                                  font=("Segoe UI", 13), width=22, show="●",
                                  highlightbackground=BORDER,
                                  highlightthickness=1)
        self.pin_entry.pack(ipady=8, padx=20, pady=(2,20))
        self.pin_entry.bind("<Return>", lambda _: self._login())

        btn = tk.Button(lc, text="➤  LOGIN",
                        bg=BLUE, fg=WHITE, relief="flat",
                        font=("Segoe UI", 12, "bold"), cursor="hand2",
                        padx=30, pady=10,
                        activebackground=CYAN, activeforeground=BG,
                        command=self._login)
        btn.pack(pady=(0,10))

        tk.Label(self, text="Demo: Account 1234567890, PIN 1234  |  by Akshay",
                 bg=BG, fg=SUBTEXT,
                 font=("Segoe UI", 8)).pack(side="bottom", pady=10)

    def _tick(self):
        self.clock_var.set(datetime.datetime.now().strftime(
            "%A, %d %B %Y  •  %I:%M:%S %p"))
        self.after(1000, self._tick)

    def _login(self):
        acc  = self.acc_entry.get().strip()
        pin  = self.pin_entry.get().strip()
        if acc not in self.db:
            messagebox.showerror("Error", "Account not found!")
            return
        if hash_pin(pin) != self.db[acc]["pin_hash"]:
            messagebox.showerror("Error", "Incorrect PIN!")
            return
        self.logged_acc = acc
        self._build_dashboard()

    # ── DASHBOARD ────────────────────────────────────────────────
    def _build_dashboard(self):
        for w in self.winfo_children():
            w.destroy()
        acc  = self.db[self.logged_acc]

        # header
        hdr = tk.Frame(self, bg=ACCENT, pady=10)
        hdr.pack(fill="x")
        tk.Label(hdr, text="🏦  NATIONAL BANK OF AKSHAY",
                 bg=ACCENT, fg=GOLD,
                 font=("Segoe UI", 16, "bold")).pack(side="left", padx=20)
        tk.Label(hdr, text=f"👤  {acc['name']}  |  {acc['account_type']} A/C",
                 bg=ACCENT, fg=CYAN,
                 font=("Segoe UI", 10, "bold")).pack(side="left", padx=10)
        tk.Button(hdr, text="🔓 Logout", bg=RED, fg=WHITE,
                  relief="flat", font=("Segoe UI", 9, "bold"),
                  cursor="hand2", padx=10, pady=4,
                  command=self._logout).pack(side="right", padx=16)
        self.clock_var = tk.StringVar()
        tk.Label(hdr, textvariable=self.clock_var,
                 bg=ACCENT, fg=SUBTEXT,
                 font=("Segoe UI", 8)).pack(side="right", padx=6)
        self._tick()

        body = tk.Frame(self, bg=BG)
        body.pack(fill="both", expand=True, padx=16, pady=12)
        body.columnconfigure(0, weight=1)
        body.columnconfigure(1, weight=2)
        body.rowconfigure(0, weight=1)

        self._build_left(body)
        self._build_right(body)

    def _build_left(self, parent):
        left = tk.Frame(parent, bg=BG)
        left.grid(row=0, column=0, sticky="nsew", padx=(0,12))

        # balance card
        bc = tk.Frame(left, bg=CARD2,
                      highlightbackground=BORDER, highlightthickness=2)
        bc.pack(fill="x", pady=(0,12))
        tk.Label(bc, text="💳  YOUR CARD", bg=CARD2, fg=SUBTEXT,
                 font=("Segoe UI", 9, "bold")).pack(anchor="w", padx=16, pady=(12,0))
        acc = self.db[self.logged_acc]
        tk.Label(bc, text=acc["card_no"], bg=CARD2, fg=TEXT,
                 font=("Courier New", 14, "bold")).pack(anchor="w", padx=16, pady=4)
        tk.Frame(bc, bg=BORDER, height=1).pack(fill="x", padx=16, pady=4)
        tk.Label(bc, text="AVAILABLE BALANCE", bg=CARD2, fg=SUBTEXT,
                 font=("Segoe UI", 9)).pack(anchor="w", padx=16)
        self.bal_var = tk.StringVar(value=f"₹{acc['balance']:,.2f}")
        tk.Label(bc, textvariable=self.bal_var, bg=CARD2, fg=GREEN,
                 font=("Segoe UI", 26, "bold")).pack(anchor="w", padx=16, pady=(0,12))

        # menu buttons
        mc = tk.Frame(left, bg=CARD,
                      highlightbackground=BORDER, highlightthickness=1)
        mc.pack(fill="x", pady=(0,12))
        tk.Label(mc, text="QUICK ACTIONS", bg=CARD, fg=CYAN,
                 font=("Segoe UI", 10, "bold")).pack(anchor="w", padx=14, pady=(10,6))
        actions = [
            ("💰  CHECK BALANCE",    GREEN,  self._check_balance),
            ("📥  DEPOSIT",          BLUE,   self._deposit),
            ("📤  WITHDRAW",         ORANGE, self._withdraw),
            ("🔁  TRANSFER",         CYAN,   self._transfer),
            ("📋  MINI STATEMENT",   ACCENT, self._statement),
            ("🔑  CHANGE PIN",       RED,    self._change_pin),
        ]
        for label, color, cmd in actions:
            b = tk.Button(mc, text=label, bg=color, fg=WHITE,
                          relief="flat", font=("Segoe UI", 11, "bold"),
                          cursor="hand2", anchor="w",
                          padx=14, pady=8,
                          activebackground=GOLD, activeforeground=BG,
                          command=cmd)
            b.pack(fill="x", padx=10, pady=3)
        tk.Frame(mc, bg=BG, height=6).pack()

    def _build_right(self, parent):
        right = tk.Frame(parent, bg=BG)
        right.grid(row=0, column=1, sticky="nsew")
        right.rowconfigure(1, weight=1)
        right.columnconfigure(0, weight=1)

        # panel title
        self.panel_title = tk.StringVar(value="WELCOME")
        tk.Label(right, textvariable=self.panel_title,
                 bg=BG, fg=CYAN,
                 font=("Segoe UI", 14, "bold")).grid(
                     row=0, column=0, sticky="w", pady=(0,8))

        # content frame
        self.content = tk.Frame(right, bg=CARD,
                                highlightbackground=BORDER,
                                highlightthickness=1)
        self.content.grid(row=1, column=0, sticky="nsew")

        self._show_welcome()

    def _clear_content(self):
        for w in self.content.winfo_children():
            w.destroy()

    def _refresh_balance(self):
        acc = self.db[self.logged_acc]
        self.bal_var.set(f"₹{acc['balance']:,.2f}")

    def _show_welcome(self):
        self._clear_content()
        self.panel_title.set("WELCOME")
        acc = self.db[self.logged_acc]
        h = datetime.datetime.now().hour
        g = "Good morning" if h<12 else ("Good afternoon" if h<17 else "Good evening")
        tk.Label(self.content,
                 text=f"{g},\n{acc['name']}!",
                 bg=CARD, fg=GOLD,
                 font=("Segoe UI", 28, "bold")).pack(pady=(40,8))
        tk.Label(self.content,
                 text=f"Account Type: {acc['account_type']}",
                 bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 11)).pack()
        tk.Label(self.content,
                 text=f"Card: {acc['card_no']}",
                 bg=CARD, fg=TEXT,
                 font=("Courier New", 12)).pack(pady=4)
        tk.Label(self.content,
                 text=f"Balance: ₹{acc['balance']:,.2f}",
                 bg=CARD, fg=GREEN,
                 font=("Segoe UI", 18, "bold")).pack(pady=8)
        tk.Label(self.content,
                 text="Select an option from the left menu.",
                 bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 10)).pack(pady=20)

    def _check_balance(self):
        self._clear_content()
        self.panel_title.set("ACCOUNT BALANCE")
        acc = self.db[self.logged_acc]
        tk.Label(self.content, text="💰  AVAILABLE BALANCE",
                 bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 12, "bold")).pack(pady=(50,8))
        tk.Label(self.content,
                 text=f"₹{acc['balance']:,.2f}",
                 bg=CARD, fg=GREEN,
                 font=("Segoe UI", 42, "bold")).pack()
        tk.Label(self.content,
                 text=f"Account: {self.logged_acc}  |  {acc['account_type']}",
                 bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 10)).pack(pady=8)
        tk.Label(self.content,
                 text=datetime.datetime.now().strftime("As of  %d %b %Y  %I:%M %p"),
                 bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 9)).pack()

    def _make_amount_form(self, title, label, on_submit, color=BLUE):
        self._clear_content()
        self.panel_title.set(title)
        tk.Label(self.content, text=label, bg=CARD, fg=TEXT,
                 font=("Segoe UI", 11)).pack(pady=(40,6))
        amt_var = tk.StringVar()
        e = tk.Entry(self.content, textvariable=amt_var,
                     bg=CARD2, fg=TEXT, insertbackground=TEXT,
                     relief="flat", font=("Segoe UI", 18, "bold"), width=16,
                     highlightbackground=BORDER, highlightthickness=1,
                     justify="center")
        e.pack(ipady=12, pady=4)
        e.focus()
        e.bind("<Return>", lambda _: on_submit(amt_var.get()))

        self.note_var = tk.StringVar()
        tk.Label(self.content, text="Note (optional):", bg=CARD,
                 fg=SUBTEXT, font=("Segoe UI", 9)).pack(pady=(10,2))
        tk.Entry(self.content, textvariable=self.note_var,
                 bg=CARD2, fg=TEXT, insertbackground=TEXT,
                 relief="flat", font=("Segoe UI", 10), width=24,
                 highlightbackground=BORDER,
                 highlightthickness=1).pack(ipady=6)

        tk.Button(self.content, text=f"✔  {title}",
                  bg=color, fg=WHITE, relief="flat",
                  font=("Segoe UI", 12, "bold"), cursor="hand2",
                  padx=20, pady=10,
                  activebackground=GOLD, activeforeground=BG,
                  command=lambda: on_submit(amt_var.get())).pack(pady=16)

    def _deposit(self):
        def do_deposit(amt_str):
            try:
                amt = float(amt_str)
                if amt <= 0: raise ValueError
            except:
                messagebox.showerror("Error", "Enter a valid amount.")
                return
            self.db[self.logged_acc]["balance"] += amt
            log_txn(self.db, self.logged_acc, "Deposit", amt,
                    self.note_var.get())
            self._refresh_balance()
            self._show_receipt("Deposit", amt, "CREDITED")
        self._make_amount_form("DEPOSIT", "Enter amount to deposit (₹):",
                               do_deposit, BLUE)

    def _withdraw(self):
        def do_withdraw(amt_str):
            try:
                amt = float(amt_str)
                if amt <= 0: raise ValueError
            except:
                messagebox.showerror("Error", "Enter a valid amount.")
                return
            if amt > self.db[self.logged_acc]["balance"]:
                messagebox.showerror("Error", "Insufficient funds!")
                return
            self.db[self.logged_acc]["balance"] -= amt
            log_txn(self.db, self.logged_acc, "Withdraw", amt,
                    self.note_var.get())
            self._refresh_balance()
            self._show_receipt("Withdrawal", amt, "DEBITED")
        self._make_amount_form("WITHDRAW", "Enter amount to withdraw (₹):",
                               do_withdraw, ORANGE)

    def _transfer(self):
        self._clear_content()
        self.panel_title.set("FUND TRANSFER")
        tk.Label(self.content, text="Recipient Account No:", bg=CARD,
                 fg=TEXT, font=("Segoe UI", 10)).pack(pady=(30,4))
        to_var = tk.StringVar()
        tk.Entry(self.content, textvariable=to_var,
                 bg=CARD2, fg=TEXT, insertbackground=TEXT,
                 relief="flat", font=("Segoe UI", 13), width=20,
                 highlightbackground=BORDER,
                 highlightthickness=1).pack(ipady=8)
        tk.Label(self.content, text="Amount (₹):", bg=CARD,
                 fg=TEXT, font=("Segoe UI", 10)).pack(pady=(12,4))
        amt_var = tk.StringVar()
        tk.Entry(self.content, textvariable=amt_var,
                 bg=CARD2, fg=TEXT, insertbackground=TEXT,
                 relief="flat", font=("Segoe UI", 18, "bold"), width=16,
                 highlightbackground=BORDER,
                 highlightthickness=1, justify="center").pack(ipady=10)
        self.note_var = tk.StringVar()
        tk.Label(self.content, text="Note:", bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 9)).pack(pady=(8,2))
        tk.Entry(self.content, textvariable=self.note_var,
                 bg=CARD2, fg=TEXT, insertbackground=TEXT,
                 relief="flat", font=("Segoe UI", 10), width=24,
                 highlightbackground=BORDER,
                 highlightthickness=1).pack(ipady=6)

        def do_transfer():
            to  = to_var.get().strip()
            try:
                amt = float(amt_var.get())
                if amt <= 0: raise ValueError
            except:
                messagebox.showerror("Error", "Enter a valid amount."); return
            if to not in self.db:
                messagebox.showerror("Error", "Recipient account not found!"); return
            if to == self.logged_acc:
                messagebox.showerror("Error", "Cannot transfer to same account!"); return
            if amt > self.db[self.logged_acc]["balance"]:
                messagebox.showerror("Error", "Insufficient funds!"); return
            self.db[self.logged_acc]["balance"] -= amt
            self.db[to]["balance"]              += amt
            note = self.note_var.get() or f"Transfer to {to}"
            log_txn(self.db, self.logged_acc, "Transfer Out", amt, note)
            log_txn(self.db, to, "Transfer In", amt,
                    f"From {self.logged_acc}")
            self._refresh_balance()
            self._show_receipt("Transfer", amt, "DEBITED",
                               extra=f"To A/C: {to}\nName: {self.db[to]['name']}")

        tk.Button(self.content, text="🔁  TRANSFER NOW",
                  bg=CYAN, fg=BG, relief="flat",
                  font=("Segoe UI", 12, "bold"), cursor="hand2",
                  padx=20, pady=10,
                  activebackground=GOLD, activeforeground=BG,
                  command=do_transfer).pack(pady=16)

    def _statement(self):
        self._clear_content()
        self.panel_title.set("MINI STATEMENT")
        acc = self.db[self.logged_acc]
        txns = acc["transactions"][-10:]
        if not txns:
            tk.Label(self.content, text="📋  No transactions yet.",
                     bg=CARD, fg=SUBTEXT,
                     font=("Segoe UI", 12)).pack(pady=50)
            return

        tk.Label(self.content, text=f"Last {len(txns)} Transactions",
                 bg=CARD, fg=SUBTEXT,
                 font=("Segoe UI", 10, "bold")).pack(pady=(12,4))

        style = ttk.Style()
        style.configure("Txn.Treeview",
                        background=CARD2, foreground=TEXT,
                        fieldbackground=CARD2, rowheight=32,
                        font=("Segoe UI", 9))
        style.configure("Txn.Treeview.Heading",
                        background=ACCENT, foreground=CYAN,
                        font=("Segoe UI", 10, "bold"), relief="flat")
        style.map("Txn.Treeview",
                  background=[("selected", BLUE)],
                  foreground=[("selected", WHITE)])

        tf = tk.Frame(self.content, bg=CARD2,
                      highlightbackground=BORDER, highlightthickness=1)
        tf.pack(fill="both", expand=True, padx=12, pady=(0,12))
        tf.rowconfigure(0, weight=1); tf.columnconfigure(0, weight=1)

        cols = ("Date", "Type", "Amount", "Balance")
        tree = ttk.Treeview(tf, columns=cols, show="headings",
                            style="Txn.Treeview", height=len(txns))
        widths = [150, 120, 100, 100]
        for col, w in zip(cols, widths):
            tree.heading(col, text=col)
            tree.column(col, width=w, anchor="center")
        tree.column("Date", anchor="w")

        for t in reversed(txns):
            col = GREEN if "In" in t["type"] or "Deposit" in t["type"] else RED
            tree.insert("", "end",
                        values=(t["time"], t["type"],
                                f"₹{t['amount']:,.2f}",
                                f"₹{t['balance']:,.2f}"),
                        tags=(t["type"],))
            tree.tag_configure(t["type"], foreground=col)
        tree.grid(row=0, column=0, sticky="nsew")

    def _change_pin(self):
        self._clear_content()
        self.panel_title.set("CHANGE PIN")
        tk.Label(self.content, text="🔑  SET NEW PIN",
                 bg=CARD, fg=CYAN,
                 font=("Segoe UI", 12, "bold")).pack(pady=(40,8))
        tk.Label(self.content, text="New PIN (4 digits):", bg=CARD,
                 fg=TEXT, font=("Segoe UI", 10)).pack(pady=(8,4))
        new1_var = tk.StringVar()
        e1 = tk.Entry(self.content, textvariable=new1_var,
                      bg=CARD2, fg=TEXT, insertbackground=TEXT,
                      relief="flat", font=("Segoe UI", 18, "bold"), width=12,
                      highlightbackground=BORDER, show="●",
                      highlightthickness=1, justify="center")
        e1.pack(ipady=10)
        e1.focus()
        tk.Label(self.content, text="Confirm PIN:", bg=CARD,
                 fg=TEXT, font=("Segoe UI", 10)).pack(pady=(12,4))
        new2_var = tk.StringVar()
        tk.Entry(self.content, textvariable=new2_var,
                 bg=CARD2, fg=TEXT, insertbackground=TEXT,
                 relief="flat", font=("Segoe UI", 18, "bold"), width=12,
                 highlightbackground=BORDER, show="●",
                 highlightthickness=1, justify="center").pack(ipady=10)

        def do_change():
            p1 = new1_var.get().strip()
            p2 = new2_var.get().strip()
            if not p1.isdigit() or len(p1) != 4:
                messagebox.showerror("Error", "PIN must be 4 digits!"); return
            if p1 != p2:
                messagebox.showerror("Error", "PINs don't match!"); return
            self.db[self.logged_acc]["pin_hash"] = hash_pin(p1)
            save_db(self.db)
            messagebox.showinfo("Success", "PIN changed successfully!")
            self._show_welcome()

        tk.Button(self.content, text="✔  CHANGE PIN",
                  bg=RED, fg=WHITE, relief="flat",
                  font=("Segoe UI", 12, "bold"), cursor="hand2",
                  padx=20, pady=10,
                  activebackground=GOLD, activeforeground=BG,
                  command=do_change).pack(pady=16)

    def _show_receipt(self, txn_type, amt, status, extra=""):
        self._clear_content()
        self.panel_title.set("TRANSACTION RECEIPT")
        acc = self.db[self.logged_acc]
        tk.Label(self.content, text="✅  TRANSACTION SUCCESSFUL",
                 bg=CARD, fg=GREEN,
                 font=("Segoe UI", 14, "bold")).pack(pady=(30,8))

        rc = tk.Frame(self.content, bg=CARD2,
                      highlightbackground=BORDER, highlightthickness=1)
        rc.pack(padx=40, pady=10)
        tk.Label(rc, text="─" * 40, bg=CARD2, fg=SUBTEXT,
                 font=("Courier New", 9)).pack()
        tk.Label(rc, text="🏦  NATIONAL BANK OF AKSHAY",
                 bg=CARD2, fg=GOLD,
                 font=("Segoe UI", 11, "bold")).pack(pady=4)
        tk.Label(rc, text="ATM Transaction Receipt",
                 bg=CARD2, fg=SUBTEXT,
                 font=("Segoe UI", 9)).pack()
        tk.Label(rc, text="─" * 40, bg=CARD2, fg=SUBTEXT,
                 font=("Courier New", 9)).pack(pady=(0,6))
        tk.Label(rc, text=f"Txn Type:     {txn_type}",
                 bg=CARD2, fg=TEXT, font=("Courier New", 10), anchor="w").pack(fill="x", padx=20)
        tk.Label(rc, text=f"Amount:       ₹{amt:,.2f}  {status}",
                 bg=CARD2, fg=RED if status=="DEBITED" else GREEN,
                 font=("Courier New", 10, "bold"), anchor="w").pack(fill="x", padx=20)
        tk.Label(rc, text=f"New Balance:  ₹{acc['balance']:,.2f}",
                 bg=CARD2, fg=TEXT, font=("Courier New", 10), anchor="w").pack(fill="x", padx=20)
        tk.Label(rc, text=f"Account:      {self.logged_acc}",
                 bg=CARD2, fg=TEXT, font=("Courier New", 9), anchor="w").pack(fill="x", padx=20)
        tk.Label(rc, text=f"Name:         {acc['name']}",
                 bg=CARD2, fg=TEXT, font=("Courier New", 9), anchor="w").pack(fill="x", padx=20)
        if extra:
            for line in extra.split("\n"):
                tk.Label(rc, text=line, bg=CARD2, fg=TEXT,
                         font=("Courier New", 9), anchor="w").pack(fill="x", padx=20)
        tk.Label(rc, text=f"Date:         {datetime.datetime.now().strftime('%d %b %Y %H:%M:%S')}",
                 bg=CARD2, fg=TEXT, font=("Courier New", 9), anchor="w").pack(fill="x", padx=20)
        tk.Label(rc, text="Ref No:       " + str(random.randint(100000,999999)),
                 bg=CARD2, fg=TEXT, font=("Courier New", 9), anchor="w").pack(fill="x", padx=20, pady=(0,6))
        tk.Label(rc, text="─" * 40, bg=CARD2, fg=SUBTEXT,
                 font=("Courier New", 9)).pack()
        tk.Label(rc, text="Thank you for banking with us!",
                 bg=CARD2, fg=SUBTEXT,
                 font=("Segoe UI", 8)).pack(pady=6)

        tk.Button(self.content, text="🏠  BACK TO HOME",
                  bg=BLUE, fg=WHITE, relief="flat",
                  font=("Segoe UI", 10, "bold"), cursor="hand2",
                  padx=16, pady=8,
                  command=self._show_welcome).pack(pady=8)

    def _logout(self):
        if messagebox.askyesno("Logout", "Are you sure you want to logout?"):
            self.logged_acc = None
            self._build_login()


if __name__ == "__main__":
    app = ATMApp()
    app.mainloop()
