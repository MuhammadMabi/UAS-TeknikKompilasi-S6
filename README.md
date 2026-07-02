# LAPORAN PROYEK AKHIR: TEKNIK KOMPILASI
## Representasi Tahapan Kompilasi pada Konstruksi `While` Loop

- **Mata Kuliah:** Teknik Kompilasi
- **Program Studi:** Teknik Informatika (Semester 6)
- **Nama Mahasiswa:** [Isi Nama]
- **NIM:** [Isi NIM]

---

# 1. PENDAHULUAN & PEMILIHAN KONSTRUKSI

Proyek akhir ini bertujuan untuk menyimulasikan dan merepresentasikan tahapan-tahapan utama dalam proses kompilasi (*compiler*), mulai dari pembacaan kode sumber (*source code*) hingga dihasilkan kode antara atau *Three-Address Code* (TAC).

Konstruksi bahasa pemrograman yang dipilih adalah **Perulangan (`while`)**. Konstruksi `while` dipilih karena memiliki alur kontrol (*control flow*) yang melibatkan pengecekan kondisi secara berulang serta proses lompatan (*conditional* dan *unconditional jump*) pada hasil TAC.

---

# 2. PATTERN (POLA SINTAKS / BNF)

```text
<while_stmt> ::= "while" "(" <condition> ")" "{" <statement> "}"

<condition> ::= <identifier> <rel_op> <value>

<statement> ::= <identifier> "=" <identifier> <math_op> <value>

<rel_op> ::= "<" | ">" | "<=" | ">=" | "==" | "!="

<math_op> ::= "+" | "-" | "*" | "/"

<identifier> ::= "a" | "b" | ... | "z"
              | kombinasi string alfanumerik

<value> ::= [0-9]+
```

---

# 3. IMPLEMENTASI PROGRAM (PYTHON)

Program diimplementasikan menggunakan bahasa **Python** dengan pendekatan **Object-Oriented Programming (OOP)** serta memanfaatkan library `re` (*Regular Expression*) untuk proses analisis leksikal.

```python
import re

class WhileCompiler:
    def __init__(self, source_code):
        self.source_code = source_code
        self.label_counter = 1

        # Symbol Table sederhana
        self.symbol_table = {
            "counter": "int",
            "limit": "int",
            "total": "int"
        }

    def new_label(self):
        lbl = f"L{self.label_counter}"
        self.label_counter += 1
        return lbl

    def lexical_analysis(self):
        """Tahap Analisis Leksikal"""

        token_specification = [
            ('WHILE', r'while'),
            ('LPAREN', r'\('),
            ('RPAREN', r'\)'),
            ('LBRACE', r'\{'),
            ('RBRACE', r'\}'),
            ('OP', r'[+\-*/=<>]'),
            ('ID', r'[a-zA-Z_][a-zA-Z0-9_]*'),
            ('NUM', r'\d+'),
            ('SKIP', r'[ \t\n]+'),
        ]

        tok_regex = '|'.join(
            f'(?P<{name}>{pattern})'
            for name, pattern in token_specification
        )

        tokens = []

        for mo in re.finditer(tok_regex, self.source_code):
            kind = mo.lastgroup
            value = mo.group()

            if kind == "SKIP":
                continue

            tokens.append((kind, value))

        return tokens

    def syntax_semantic_analysis(self, tokens):
        """Tahap Sintaksis & Semantik"""

        try:
            token_kinds = [t[0] for t in tokens]

            if token_kinds[0] != "WHILE":
                raise SyntaxError("Harus diawali keyword while")

            lparen = token_kinds.index("LPAREN")
            rparen = token_kinds.index("RPAREN")

            lbrace = token_kinds.index("LBRACE")
            rbrace = token_kinds.index("RBRACE")

            condition = tokens[lparen+1:rparen]
            body = tokens[lbrace+1:rbrace]

        except (ValueError, IndexError):
            raise SyntaxError("Struktur while tidak valid")

        for kind, value in tokens:
            if kind == "ID":
                if value not in self.symbol_table:
                    raise NameError(
                        f"Variabel {value} belum dideklarasikan"
                    )

        condition_str = " ".join([t[1] for t in condition])
        body_str = " ".join([t[1] for t in body])

        return condition_str, body_str

    def generate_tac(self):
        tokens = self.lexical_analysis()

        condition, body = self.syntax_semantic_analysis(tokens)

        label_start = self.new_label()
        label_end = self.new_label()

        tac = []

        tac.append(f"{label_start}:")
        tac.append(f"ifFalse {condition} goto {label_end}")
        tac.append(f"    {body}")
        tac.append(f"goto {label_start}")
        tac.append(f"{label_end}:")

        return tokens, tac
```

---

# 4. PENJELASAN TAHAPAN KOMPILASI

## A. Analisis Leksikal (Lexical Analysis)

Tahap ini bertugas membaca kode sumber dan memecahnya menjadi token-token.

Input:

```text
while ( counter < limit ) { total = total + 5 }
```

Output token:

```text
('WHILE', 'while')
('LPAREN', '(')
('ID', 'counter')
('OP', '<')
('ID', 'limit')
('RPAREN', ')')
('LBRACE', '{')
('ID', 'total')
('OP', '=')
('ID', 'total')
('OP', '+')
('NUM', '5')
('RBRACE', '}')
```

---

## B. Analisis Sintaksis (Syntax Analysis)

Tahap ini memeriksa apakah urutan token telah sesuai dengan aturan grammar (BNF).

Beberapa validasi yang dilakukan:

- Program harus diawali keyword `while`
- Memastikan pasangan `(` dan `)`
- Memastikan pasangan `{` dan `}`
- Memisahkan bagian kondisi dan isi perulangan

Jika ada kesalahan struktur, program akan menghasilkan `SyntaxError`.

---

## C. Analisis Semantik (Semantic Analysis)

Setelah sintaks benar, compiler memeriksa makna program.

Pada simulasi ini dilakukan pengecekan:

- Apakah setiap variabel telah terdaftar pada **Symbol Table**
- Jika variabel belum dideklarasikan maka muncul `NameError`.

Contoh Symbol Table:

```text
counter : int
limit   : int
total   : int
```

---

## D. Generasi Kode Antara (Three-Address Code)

Tahap terakhir menghasilkan **Three-Address Code (TAC)**.

Flow TAC:

- **L1** → awal perulangan
- Cek kondisi
- Jika kondisi salah lompat ke **L2**
- Jalankan isi perulangan
- Kembali ke **L1**
- **L2** → akhir perulangan

---

# 5. HASIL EKSEKUSI PROGRAM

Input:

```text
while ( counter < limit ) { total = total + 5 }
```

Output:

```text
==================================================
INPUT SOURCE CODE
while ( counter < limit ) { total = total + 5 }
==================================================

1. ANALISIS LEKSIKAL

('WHILE', 'while')
('LPAREN', '(')
('ID', 'counter')
('OP', '<')
('ID', 'limit')
('RPAREN', ')')
('LBRACE', '{')
('ID', 'total')
('OP', '=')
('ID', 'total')
('OP', '+')
('NUM', '5')
('RBRACE', '}')

--------------------------------------------------

2. ANALISIS SINTAKSIS

Status : Valid

--------------------------------------------------

3. ANALISIS SEMANTIK

Status : Semua variabel ditemukan pada Symbol Table

--------------------------------------------------

4. THREE ADDRESS CODE

L1:
ifFalse counter < limit goto L2
    total = total + 5
goto L1
L2:
==================================================
```

---

# 6. KESIMPULAN

Melalui implementasi ini dapat disimpulkan bahwa proses kompilasi terdiri dari beberapa tahapan penting, yaitu **Analisis Leksikal**, **Analisis Sintaksis**, **Analisis Semantik**, dan **Generasi Kode Antara (Three-Address Code)**. Program yang dibuat berhasil mensimulasikan setiap tahapan tersebut pada konstruksi **`while` loop**, mulai dari pemecahan token, pemeriksaan struktur, validasi variabel menggunakan *Symbol Table*, hingga menghasilkan TAC sebagai representasi kode antara sebelum diterjemahkan ke bahasa mesin.
