# Laporan Praktikum Proporsional Logic KKA A
---

| Nama | NRP | Kelas |
| :--- | :--- | :--- |
| Raymond Julius Pardosi | 5025241268 | A |


---

### **Bab 1: Pendahuluan**
<p style="text-align: justify;">
Kecerdasan Buatan (<i>Artificial Intelligence</i>) - AI merupakan bidang ilmu yang bertujuan untuk menciptakan agen cerdas yang mampu berinteraksi dengan lingkungannya, membuat keputusan, dan memecahkan masalah. Dalam konteks AI, <i>Proporsional Logic</i> merupakan fundamental kerja logika mesin untuk merepresentasikan pengetahuan dan melakukan keputusan logis. Praktikum ini bertujuan untuk mengeksplorasi dan mengimplementasikan logika AI menggunakan <i>Knowledge Based System</i>.
</p>

---

### **Bab 2: Pembahasan**
Praktikum ini akan menggunakan beberapa library dan modul. Berikut adalah list import yang akan dilakukan:

```
%pip install ipythonblocks
%pip install qpsolvers
from utils  *
from logic import *
from notebook import psource
```


Selanjutnya, bagian ini membahas secara mendalam tentang <i>Logical Sentences</i>, termasuk:


**2.1. EXPR:**

<p style="text-align: justify;">
Expr merupakan class untuk merepresentasi ekspresi matematika sehingga operasi matematika, seperti simbol dan operator, dapat didefinisikan menggunakan class expr. Berikut merupakan beberapa operator:
</p>

| Operation | Book  | Python Infix Input | Python Output | Python `Expr` Input |
| :--- | :--- | :--- | :--- | :--- |
| **Negation** | &not; P | `~P` | `~P` | `Expr('~', P)` |
| **And** | P &and; Q | `P & Q` | `P & Q` | `Expr('&', P, Q)` |
| **Or** | P &or; Q | `P \| Q` | `P \| Q` | `Expr('\|', P, Q)` |
| **Inequality (Xor)** | P &ne; Q | `P ^ Q` | `P ^ Q` | `Expr('^', P, Q)` |
| **Implication** | P &rarr; Q | `P ==> Q` | `P ==> Q` | `Expr('==>', P, Q)` |
| **Reverse Implication** | Q &larr; P | `Q <== P` | `Q <== P` | `Expr('<==', Q, P)` |
| **Equivalence** | P &harr; Q | `P <=> Q` | `P <=> Q` | `Expr('<=>', P, Q)` |

Contoh penggunaan expr dalam fungsinya:
<br>

    Input: 
    expr('sqrt(b ** 2 - 4 * a * c)')
    
    Output:
    sqrt(((b ** 2) - ((4 * a) * c)))
</br>

**2.2. Propotional Knowledge Bases (PropKB)**

<p style="text-align: justify;">
Class PropKB merupakan class python yang berfungsi untuk merepresentasikan sebuah knowledge base dari proporsional logic sentences.
</p>

Class PropKB memiliki empat buah metode, yaitu:

    __init__(self, sentence=None): 

    tell(self, sentence)

    ask_generator(self, query)
    
    retract(self, sentence)
</br>

Salah satu contoh penerapan PropKB adalah puzzle Wumpus. Contoh kasus puzzle wumpus:

    wumpus_kb = PropKB()
    
    P11, P12, P21, P22, P31, B11, B21 = expr('P11, P12, P21, P22, P31, B11, B21')

    wumpus_kb.tell(~P11)

    wumpus_kb.tell(B11 | '<=>' | ((P12 | P21)))
    wumpus_kb.tell(B21 | '<=>' | ((P11 | P22 | P31)))

    wumpus_kb.tell(~B11)
    wumpus_kb.tell(B21)

    wumpus_kb.clauses
Kasus tersebut mendefinisikan world wumpus beserta KB-nya. World wumpusnya berupa:
* No pit [1, 1]
* Breeze hanya ada ketika ada pit di square tetangga
* Breeze ada di [1, 1] dan [2, 1]
* No breeze [1, 1]

Maka di dapatkan state world seperti berikut:

![alt text](image.png)


**2.3. Knowledge-based agents**

<p style="text-align: justify;">
KB-Agent adalah sebuah mesin yang menjaga dan menangani sebuah KB. KB-Agent berfungsi untuk menyediakan abstraksi di atas manipulasi pengetahuan sehingga agen hanya perlu memanggil metode KB untuk mencari tahu tindakan apa yang harus diambil.
</p>

Contoh penerapan KB-Agent adalah menggunakan program KB_AgentProgram.
<br>
KB_AgentProgram:
```
def KB_AgentProgram(KB):
    steps = itertools.count()

    def program(percept):
        t = next(steps)

        KB.tell(make_percept_sentence(percept, t)) 

        action = KB.ask(make_action_query(t))

        KB.tell(make_action_sentence(action, t)) 

        return action

    return program
```

Setiap function dalam program tersebut berfungsi untuk:

    make_percept_sentence(percept, t): Mencatat apa yang dilihat agen pada waktu t.

    make_action_query(t): Mengajukan kueri ke KB: "Tindakan apa yang harus dilakukan pada waktu t?"

    make_action_sentence(action, t): Mencatat apa yang telah dilakukan agen pada waktu t (mengubah status KB).
Dengan struktur ini, program agen dapat berinteraksi dengan lingkungan secara berkelanjutan, terus-menerus memperbarui pengetahuannya dan mengambil keputusan yang didasarkan pada semua informasi historis yang tersimpan dalam Knowledge Base.

**2.4. Inference in Propositional Knowledge Base**

Pada bagian ini, pengecekan apakah  $\text{KB} \models \alpha$ untuk beberapa sentence $\alpha$ akan dilakukan.

Algoritma yang dapat digunakan adalah:
* Truth Table Enumeration: Komputasi seluruh barisan truth $\text{KB} \models \alpha$ dan cek kondisi truthnya.
* Proof by Resolution: Menggunakan teknik Pembuktian melalui Kontradiksi (Proof by Contradiction).
* Forward and Backward Chaining: Terbatas untuk KB yang hanya mengandung Horn Clauses (disjungsi literal dengan paling banyak satu literal positif, atau definite clauses).

**2.4.1. Truth Table Enumeration**

Program Truth Table Enumeration dapat berupa:

    def tt_check_all(kb, alpha, symbols, model):
        if not symbols:
            if pl_true(kb, model):
                result = pl_true(alpha, model)
                assert result in (True, False)
                return result
          else:
            return True
      else:
           P, rest = symbols[0], symbols[1:]
            return (tt_check_all(kb, alpha, rest, extend(model, P, True)) and
                    tt_check_all(kb, alpha, rest, extend(model, P, False)))
    
    def tt_entails(kb, alpha):
        assert not variables(alpha)
        symbols = list(prop_symbols(kb & alpha))
        return tt_check_all(kb, alpha, symbols, {})
Algoritma ini akan mengiterasikan seluruh truth $\text{KB} \models \alpha$. Fungsi tt_check_all akan evaluasi logical expression untuk setiap `model
pl_true(kb, model) => pl_true(alpha, model)` yang ekivalen dengan `pl_true(kb, model) & ~pl_true(alpha, model)` yang merupakan KB dan negasi setiap query yang tidak konsisten. 

Contoh Penerapan:
* Inisiasi simbol & entail truth value:
![alt text](image-4.png)

**2.4.2. Proof by Resolution**

Algoritma ini membuktikan $\text{KB} \models \alpha$ dengan menunjukkan bahwa $\text{KB} \land \neg \alpha$ tidak dapat dipuaskan (unsatisfiable). Algoritma ini menggunakan Conjunctive Normal Form (CNF).

Program algoritma berupa:

    def to_cnf(s):
        s = expr(s)
        if isinstance(s, str):
            s = expr(s)
        s = eliminate_implications(s)  # Steps 1, 2 from p. 253
        s = move_not_inwards(s)      # Step 3
        return distribute_and_over_or(s) # Step 4

    def eliminate_implications(s):
        s = expr(s)
        if not s.args or is_symbol(s.op):
            return s
        args = list(map(eliminate_implications, s.args))
        a, b = args[0], args[-1]
        if s.op == '==>':
            return b | ~a
        elif s.op == '<==':
            return a | ~b
        elif s.op == '<=>':
            return (a | ~b) & (b | ~a)
        elif s.op == '^':
            assert len(args) == 2
            return (a & ~b) | (~a & b)
        else:
            assert s.op in ('&', '|', '~')
            return Expr(s.op, *args)

    def move_not_inwards(s):
        s = expr(s)
        if s.op == '~':
            def NOT(b):
                return move_not_inwards(~b)
            a = s.args[0]
            if a.op == '~':
                return move_not_inwards(a.args[0])  # ~~A ==> A
            if a.op == '&':
                return associate('|', list(map(NOT, a.args)))
            if a.op == '|':
                return associate('&', list(map(NOT, a.args)))
            return s
        elif is_symbol(s.op) or not s.args:
            return s
        else:
            return Expr(s.op, *list(map(move_not_inwards, s.args)))

    def distribute_and_over_or(s):
        s = expr(s)
        if s.op == '|':
            s = associate('|', s.args)
            if s.op != '|':
                return distribute_and_over_or(s)
            if len(s.args) == 0:
                return False
            if len(s.args) == 1:
                return distribute_and_over_or(s.args[0])
            conj = first(arg for arg in s.args if arg.op == '&')
            if not conj:
                return s
            others = [a for a in s.args if a is not conj]
            rest = associate('|', others)
            return associate('&', [distribute_and_over_or(c | rest)
                                   for c in conj.args])
        elif s.op == '&':
            return associate('&', list(map(distribute_and_over_or, s.args)))
        else:
            return s

    def pl_resolution(KB, alpha):
        clauses = KB.clauses + conjuncts(to_cnf(~alpha))
        new = set()
        while True:
            n = len(clauses)
            pairs = [(clauses[i], clauses[j])
                     for i in range(n) for j in range(i+1, n)]
            for (ci, cj) in pairs:
                resolvents = pl_resolve(ci, cj)
                if False in resolvents:
                    return True
                new = new.union(set(resolvents))
            if new.issubset(set(clauses)):
                return False
            for c in new:
                if c not in clauses:
                    clauses.append(c)
Alur kerja algoritma ini adalah:
* Inisialisasi: Semua kalimat dalam $\text{KB}$ dan negasi dari kueri ($\neg \alpha$) diubah menjadi Klausa (bentuk CNF) dan dimasukkan ke dalam satu daftar.
* Iterasi Resolusi: Iterasi secara berulang proses pemilihan pasangan klausa dan menciptakan klausa baru. Klausa baru ditambahkan ke KB.
* Stop state: Jika proses resolusi menghasilkan klausa kosong ($\emptyset$), ini adalah kontradiksi (setara dengan False). Ini membuktikan bahwa $\text{KB} \land \neg \alpha$ tidak dapat dipuaskan, sehingga $\text{KB} \models \alpha$ adalah True. Jika tidak ada lagi klausa baru yang dapat dihasilkan, dan klausa kosong belum ditemukan, maka $\text{KB}$ tidak meng-entail $\alpha$ (yaitu, $\text{KB} \land \neg \alpha$ dapat dipuaskan).

Contoh penerapan:

* Inisiasi CNF
![alt text](image-1.png)
* Iterasi Resolusi dan hasil:
![alt text](image-2.png)

**2.4.3. Forward and backward chaining**

Algoritma ini akan menentukan knowledge base dari definite clause yang entails proposisi tunggal q (query). Algoritma ini hanya akan bekerja dengan knowledge base yang memiliki horn clauses (maksimal 1 literal positif).

Alur kerja algoritma:

     def clauses_with_premise(self, p):
        return [c for c in self.clauses
                if c.op == '==>' and p in conjuncts(c.args[0])]

    def pl_fc_entails(KB, q):
        count = {c: len(conjuncts(c.args[0]))
                for c in KB.clauses
                if c.op == '==>'}
        inferred = defaultdict(bool)
        agenda = [s for s in KB.clauses if is_prop_symbol(s.op)]
        while agenda:
            p = agenda.pop()
            if p == q:
                return True
            if not inferred[p]:
                inferred[p] = True
                for c in KB.clauses_with_premise(p):
                    count[c] -= 1
                    if count[c] == 0:
                        agenda.append(c.args[1])
        return False
Algoritma menggunakan KB yang definit (derived dari PropKB) untuk menyimpan klausa definit. PropDefiniteKB akan menyediakan list klausa yang memiliki symbol p dalam premisnya. Selanjutnya algoritma akan melakukan pop simbol p pada setiap interasi agenda.

Contoh penerapan:
![alt text](image-5.png)

### **Bab 3: Kesimpulan**
---
<p style="text-align: justify;">
Logika Proposisional berfungsi sebagai kerangka kerja fundamental yang memungkinkan Knowledge-Based Agent (KB-Agent) membuat keputusan logis dan sistematis di lingkungan yang tidak pasti, seperti dalam kasus Wumpus World. Keputusan tersebut dapat tercapai karena penggunaan class Expr dan PropKB untuk mengubah kalimat, klausa, dan operator menjadi sebuah data yang dapat diproses dan kelola secara sistematis. Dalam operasinya, agen akan menggunakan metode tell serta ask untuk menambahkan pengetahuan baru ke KB dan melakukan inferensi untuk menentukan solusi optimal.
</p>
