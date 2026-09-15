# OOPs Interview Prep — From Basics, in C++

A ground-up guide built around the 50 questions in the Edureka "OOPs Interview Questions" article, but answered the way a **C++** interviewer expects. Everything here compiles with `g++ -std=c++17`.

> **Read this first.** The source article is written with a Java/C# mindset. Several of its questions (`finalize`, garbage collection, `interface`, `final` variables) describe things C++ either does differently or does not have at all. Those gaps are exactly where interviews get interesting — each one is flagged below with a **⚠️ Java-ism** note and the correct C++ answer. Knowing *why* C++ made a different choice is worth more than memorising either language's version.

## How to use this file

1. **Part 1–7** teach each concept from zero. Read in order; each builds on the last.
2. **Part 8** is the rapid-fire Q&A — all 50 article questions with short, sayable answers.
3. **Part 9** is the cheat sheet and the traps interviewers set. Skim this the night before.
4. Type the code out. Don't read it. `g++ -std=c++17 -Wall file.cpp && ./a.out`.

---

# Part 0 — What OOP actually is

## Procedural (SOP) vs Object-Oriented (OOP)

**Structured/procedural programming** organises a program around *functions* that operate on *data passed to them*. The data is inert and public; the functions are the program. C, and most beginner code, works this way.

```cpp
// Procedural: data and the code that guards it are separate
struct Account { double balance; };

void withdraw(Account& a, double amt) {
    if (amt <= a.balance) a.balance -= amt;
}

int main() {
    Account a{100.0};
    a.balance = -5000.0;   // nothing stops this. The invariant is unguarded.
}
```

**Object-oriented programming** organises a program around *objects* that own their data and expose only the operations that keep that data valid. The invariant ("balance is never negative") lives *with* the balance.

```cpp
class Account {
    double balance_;                               // nobody outside can touch this
public:
    explicit Account(double opening) : balance_(opening) {}
    void withdraw(double amt) {
        if (amt > balance_) throw std::runtime_error("insufficient funds");
        balance_ -= amt;
    }
    double balance() const { return balance_; }
};
// a.balance_ = -5000.0;  // compile error. The invariant is now enforced.
```

| | Procedural (SOP) | Object-Oriented (OOP) |
|---|---|---|
| Unit of design | Function | Object (data + behaviour) |
| Data | Global / passed around, exposed | Owned by the object, hidden |
| Top-down or bottom-up | Top-down decomposition | Bottom-up composition |
| Adding a new *type* | Edit every function's `switch` | Add one new class |
| Adding a new *operation* | Add one function | Edit every class |
| Access control | None | `public` / `protected` / `private` |
| Examples | C, Pascal, Fortran | C++, Java, C#, Python |

> **Interview-grade nuance:** OOP isn't universally better — it's better at *adding types*, worse at *adding operations*. That trade-off has a name: the **Expression Problem**. Saying this out loud signals you've thought past the textbook.

## Why use OOP?

- **Encapsulation → fewer bugs.** Invalid states become unrepresentable rather than merely discouraged.
- **Reusability.** Inheritance and composition let you build on existing, tested code.
- **Extensibility.** New behaviour arrives as a new class, not as edits scattered through existing code (Open/Closed Principle).
- **Maintainability.** A change to a class's internals stops at the class boundary if the public interface holds.
- **Models the problem domain.** `Invoice`, `Customer`, `Shipment` map to nouns the business already uses.

## The four pillars

| Pillar | One-line definition | C++ mechanism |
|---|---|---|
| **Encapsulation** | Bundle data with the code that operates on it, and hide the data. | `class`, access specifiers, getters/setters |
| **Abstraction** | Expose *what* an object does, hide *how*. | Abstract base classes, pure virtual functions, headers |
| **Inheritance** | A new class acquires the members of an existing one. | `class D : public B` |
| **Polymorphism** | One interface, many behaviours. | Overloading & templates (compile time); `virtual` (runtime) |

Some lists add **Message passing** (objects communicate by calling each other's methods) — mention it only if the interviewer is clearly working from a five-pillar list.

---

# Part 1 — Classes and Objects

## Class vs Object

A **class** is a blueprint: a user-defined type describing what data an instance holds and what it can do. It occupies no memory at runtime (the *type* does not; its instances do).

An **object** is an instance of a class: an actual region of memory holding real values, created at runtime.

> Class is the *cookie cutter*, object is the *cookie*. One cutter, many cookies, each with its own dough.

```cpp
#include <iostream>
#include <string>

class Car {                       // ---- the blueprint ----
    std::string model_;           // data members (state)
    int speed_;
public:
    Car(std::string model, int speed)     // constructor
        : model_(std::move(model)), speed_(speed) {}

    void accelerate(int by) { speed_ += by; }          // member function (behaviour)
    void print() const {
        std::cout << model_ << " @ " << speed_ << " km/h\n";
    }
};

int main() {
    Car a("Swift", 60);           // ---- objects ----
    Car b("Civic", 80);           // independent state
    a.accelerate(20);
    a.print();                    // Swift @ 80 km/h
    b.print();                    // Civic @ 80 km/h
}
```

**Three ways to create objects in C++** — know the difference, it comes up constantly:

```cpp
Car onStack("A", 10);                             // automatic storage; destroyed at scope exit
Car* raw = new Car("B", 20);  delete raw;         // free store; YOU must delete. Avoid.
auto owned = std::make_unique<Car>("C", 30);      // free store, RAII-owned. Prefer this.
```

## Class vs Structure

In **C++ the only difference is the default access specifier**: `struct` members and base classes default to `public`, `class` defaults to `private`. Everything else — constructors, destructors, inheritance, virtual functions, templates, access control — works identically.

```cpp
struct P { int x; };            // x is public
class  Q { int x; };            // x is private
```

**Convention** (what to say after the technical answer): use `struct` for passive aggregates with no invariant to protect (`struct Point { double x, y; };`), use `class` when the type maintains an invariant and therefore needs to hide state.

⚠️ **If the interviewer came from C or Java:** in *C*, a struct cannot have member functions or access control at all. In *Java/C#*, `struct` (C#) is a value type vs `class` a reference type — a much bigger distinction. C++'s distinction is the small one. Say which language you're answering for.

## Data members and member functions

```cpp
class Widget {
    int id_;                                  // non-static data member: one per object
    static int count_;                        // static data member: ONE for the whole class
    mutable int cacheHits_ = 0;               // modifiable even in a const member function
public:
    Widget(int id) : id_(id) { ++count_; }
    ~Widget() { --count_; }

    int id() const { return id_; }            // const member fn: promises not to modify *this
    static int count() { return count_; }     // static member fn: no `this`, callable on the class
};
int Widget::count_ = 0;                       // static members need a definition (pre-C++17)
```

- A **`const` member function** can be called on a `const` object and cannot modify non-`mutable` members. Mark every member function `const` that doesn't mutate — interviewers notice when you don't.
- **`this`** is an implicit pointer to the current object available in every non-static member function. `this->x` and `x` are the same thing; `*this` is the object itself (used for `return *this;` in chained setters and in `operator=`).

## "Can you call a base class method without creating an instance?"

Yes — in three situations:

1. **The method is `static`.** Static members belong to the class, not to any object.
   ```cpp
   struct Base { static void ping() { std::cout << "pong\n"; } };
   Base::ping();                  // no object anywhere
   ```
2. **You have a derived object.** A derived object *contains* a base sub-object, so calling `Base::greet()` through it requires no separate `Base` instance.
   ```cpp
   struct Base    { void greet() { std::cout << "hi from Base\n"; } };
   struct Derived : Base {};
   Derived d;  d.greet();                 // Base's method, no standalone Base object
   Derived e;  e.Base::greet();           // explicit qualification, bypasses any override
   ```
3. **The class is abstract and you call it via a derived object** — a pure-virtual class can still have a *defined* implementation you call as `Base::f()`.

You **cannot** call a non-static member function with no object at all: it needs a `this`.

---

# Part 2 — Encapsulation

**Encapsulation** = bundling data and the functions that operate on it into one unit, *and* restricting direct access to that data from outside.

Two halves, and people forget the first:
1. **Bundling** — the data and its operations live together in one class.
2. **Data hiding** — the data is `private`; the outside world goes through a controlled interface.

Why it matters: it lets you **change the implementation without breaking callers**, and it gives you **one place** to enforce every invariant.

```cpp
#include <stdexcept>

class Temperature {
    double celsius_;                                  // representation — an implementation detail
public:
    explicit Temperature(double c) { setCelsius(c); } // reuse the validating setter

    double celsius() const { return celsius_; }
    double fahrenheit() const { return celsius_ * 9.0 / 5.0 + 32.0; }  // derived, not stored

    void setCelsius(double c) {
        if (c < -273.15) throw std::out_of_range("below absolute zero");
        celsius_ = c;                                 // ONE place enforces the invariant
    }
};
```

Note `fahrenheit()` — from outside, a caller can't tell whether it's stored or computed. That freedom *is* encapsulation. If `celsius_` were public, switching the internal representation to Kelvin would break every caller.

> **Trap:** "Encapsulation means writing getters and setters for every member." No. A public setter for every private member is just a slower public member. Encapsulation means exposing *operations meaningful to the domain* (`withdraw`, `deposit`) rather than raw state.

## Access specifiers

An **access specifier** controls which code may name a member.

| Specifier | Accessible from | Typical use |
|---|---|---|
| `public` | Anywhere | The interface. What callers may use. |
| `protected` | The class itself and its derived classes | Members subclasses need but outsiders must not touch |
| `private` | The class itself and its `friend`s only | Everything else — the default for `class` |

```cpp
class Base {
public:    int pub = 1;
protected: int prot = 2;
private:   int priv = 3;

    friend void peek(const Base&);        // friends get access to everything
};

void peek(const Base& b) { (void)b.priv; }  // legal: peek is a friend

class Derived : public Base {
    void f() {
        pub  = 10;    // OK
        prot = 20;    // OK — that's what protected is for
        // priv = 30; // ERROR — private is not inherited-accessible
    }
};

int main() {
    Base b;
    b.pub = 1;        // OK
    // b.prot = 2;    // ERROR
    // b.priv = 3;    // ERROR
}
```

Key points to land:
- Access is checked **per class, not per object**: a `Base` member function can touch the `private` members of *any* `Base`, not just its own.
- `protected` is a design smell in large hierarchies — it exposes state to an unbounded set of future subclasses. Prefer `private` + a `protected` (or `private`) virtual hook.
- `friend` is not a violation of encapsulation; it's part of the class's declared interface. The class chooses its friends.

## Inheritance access modes (C++-only, and a favourite question)

C++ additionally lets you specify *how* the base's members appear in the derived class:

```cpp
class D1 : public    Base {};   // public stays public, protected stays protected  → "is-a"
class D2 : protected Base {};   // public AND protected become protected           → rare
class D3 : private   Base {};   // public AND protected become private             → "implemented-in-terms-of"
```

| Base member | `public` inheritance | `protected` inheritance | `private` inheritance |
|---|---|---|---|
| `public` | public | protected | private |
| `protected` | protected | protected | private |
| `private` | inaccessible | inaccessible | inaccessible |

- **`public`** is the only one that models **"is-a"** — only it allows implicit conversion from `Derived*` to `Base*`.
- **`private`** models **"is implemented in terms of"**. Composition usually does this better and more simply.
- Default is `private` for `class`, `public` for `struct` — a classic gotcha: `class D : Base {}` is *private* inheritance, and then `D* → Base*` silently won't compile.

---

# Part 3 — Abstraction

**Abstraction** = exposing only the essential characteristics of an entity and hiding the implementation details. You describe *what* something does, never *how*.

You use it every day: `std::sort` sorts; you don't know or care that it's an introsort. `std::vector::push_back` appends; the reallocation strategy is invisible.

## Abstraction vs Encapsulation — the question everyone fumbles

They are related but distinct. Say it like this:

| | Abstraction | Encapsulation |
|---|---|---|
| **Concern** | Design: *what* to expose | Implementation: *how* to hide it |
| **Question answered** | "What operations does this type offer?" | "How do I stop callers from corrupting state?" |
| **Level** | Interface / design level | Code / implementation level |
| **C++ mechanism** | Abstract classes, pure virtual functions, header/impl split, templates | `private`/`protected`, getters/setters, `friend`, the Pimpl idiom |
| **Solves** | Complexity — you needn't understand the whole system to use a part | Integrity + change-isolation |

One sentence to memorise: **Abstraction is about hiding complexity; encapsulation is about hiding data.** Abstraction decides what the interface *is*; encapsulation is the enforcement that keeps callers on that interface.

## How to achieve abstraction in C++

1. **Abstract base classes with pure virtual functions** (the classic OOP way).
2. **Access specifiers** — `private` implementation, `public` interface.
3. **Header / source separation** — callers see declarations, not definitions.
4. **The Pimpl idiom** — hide the data members themselves behind an opaque pointer.
5. **Templates & concepts** — abstraction over *types* without inheritance.

## Abstract classes and pure virtual functions

A **pure virtual function** is a virtual function declared `= 0`: it has no (required) implementation and *must* be overridden by a concrete derived class.

An **abstract class** is any class with at least one pure virtual function. It **cannot be instantiated** — but you can absolutely have pointers and references to it, which is the entire point.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Shape {                                   // abstract base class
public:
    virtual double area() const = 0;            // pure virtual: subclasses MUST define
    virtual void describe() const {             // ordinary virtual: default provided, overridable
        std::cout << "A shape of area " << area() << "\n";
    }
    virtual ~Shape() = default;                 // ALWAYS: virtual dtor in a polymorphic base
};

class Circle : public Shape {
    double r_;
public:
    explicit Circle(double r) : r_(r) {}
    double area() const override { return 3.14159265 * r_ * r_; }
};

class Rect : public Shape {
    double w_, h_;
public:
    Rect(double w, double h) : w_(w), h_(h) {}
    double area() const override { return w_ * h_; }
    void describe() const override {
        std::cout << "Rect " << w_ << "x" << h_ << " area " << area() << "\n";
    }
};

int main() {
    // Shape s;                                  // ERROR: cannot instantiate an abstract class
    std::vector<std::unique_ptr<Shape>> shapes;  // but pointers to it are fine — and essential
    shapes.push_back(std::make_unique<Circle>(2.0));
    shapes.push_back(std::make_unique<Rect>(3.0, 4.0));

    for (const auto& s : shapes) s->describe();  // runtime polymorphism
}
```

**"Can you create an instance of an abstract class?"** — No. The compiler rejects it, because the object would have pure virtual functions with nothing to dispatch to. You *can* declare pointers/references to it, and derived objects are created normally (their construction runs the abstract base's constructor as a sub-object).

**Subtlety worth mentioning:** a pure virtual function *may* still have a body, which derived classes call explicitly. Useful for providing a default that subclasses must opt into:

```cpp
class Logger {
public:
    virtual void log(const std::string& m) = 0;
    virtual ~Logger() = default;
};
inline void Logger::log(const std::string&) { /* shared default behaviour */ }
// A derived class writes:  void log(const std::string& m) override { Logger::log(m); ... }
```

A **pure virtual destructor** is the way to make a class abstract when it has no other suitable function — and it *must* have a body, because derived destructors call it:

```cpp
struct AbstractBase { virtual ~AbstractBase() = 0; };
AbstractBase::~AbstractBase() = default;      // required
```

## Interfaces in C++

⚠️ **Java-ism.** C++ has **no `interface` keyword**. The equivalent is a class where *every* function is pure virtual and there is no state:

```cpp
class Drawable {                 // "interface" by convention
public:
    virtual void draw() const = 0;
    virtual ~Drawable() = default;
};
```

Because C++ supports **multiple inheritance**, a class can implement many such interfaces — which is precisely why Java needed a separate `interface` construct (it allows multiple *interface* inheritance while banning multiple *implementation* inheritance, to dodge the diamond problem).

| | Abstract class (C++) | Interface (pure ABC) |
|---|---|---|
| Can hold data members | Yes | No, by convention |
| Can define some methods | Yes | No — all pure virtual |
| Purpose | Share partial implementation up a hierarchy | Declare a contract / capability |
| Relationship | "is-a" | "can-do" |
| Multiple inheritance | Possible but risky (diamond) | Safe and idiomatic |

**Modern alternative worth one sentence:** in C++20, **concepts** give you compile-time interfaces with zero runtime cost — no vtable, no pointer indirection. `template<typename T> concept Drawable = requires(const T& t) { t.draw(); };`

---

# Part 4 — Inheritance

**Inheritance** lets a new class (**derived / subclass / child**) acquire the members of an existing class (**base / superclass / parent**), then extend or specialise them. It models an **"is-a"** relationship and enables code reuse plus runtime polymorphism.

- **Superclass / base / parent**: the class being inherited *from*. Generalisation.
- **Subclass / derived / child**: the class that inherits. Specialisation — it has everything the base has, plus more.

```cpp
#include <iostream>
#include <string>

class Employee {                                   // base / superclass
protected:
    std::string name_;
    double base_;
public:
    Employee(std::string n, double b) : name_(std::move(n)), base_(b) {}
    virtual double salary() const { return base_; }
    virtual ~Employee() = default;
    const std::string& name() const { return name_; }
};

class Manager : public Employee {                  // derived / subclass — "a Manager IS-A Employee"
    double bonus_;
public:
    Manager(std::string n, double b, double bonus)
        : Employee(std::move(n), b), bonus_(bonus) {}      // base ctor in the init list
    double salary() const override { return base_ + bonus_; }   // specialised
};
```

## Order of construction and destruction

Memorise this; it is asked constantly.

- **Construction:** base class → derived's data members (in *declaration order*, not init-list order) → derived constructor body.
- **Destruction:** exactly the reverse — derived destructor body → derived's members → base destructor.

```cpp
struct A { A(){std::cout<<"A ";} ~A(){std::cout<<"~A ";} };
struct M { M(){std::cout<<"M ";} ~M(){std::cout<<"~M ";} };
struct B : A { M m; B(){std::cout<<"B ";} ~B(){std::cout<<"~B ";} };
// B b;  prints:  A M B      then at scope exit:  ~B ~M ~A
```

> **Trap:** calling a virtual function from a constructor or destructor does **not** dispatch to the derived override — during base construction the object *is not yet* a derived object, so the base version runs. (Java does the opposite and calls the override on a half-initialised object. Both are bugs waiting to happen; C++'s is at least predictable.)

## Types of inheritance

```
1. Single            2. Multilevel        3. Hierarchical      4. Multiple          5. Hybrid
   A                    A                    A                  A   B                  A
   |                    |                   / \                  \ /                  / \
   B                    B                  B   C                  C                  B   C
                        |                                                              \ /
                        C                                                               D
```

| Type | Shape | C++ |
|---|---|---|
| **Single** | One base, one derived | `class B : public A {};` |
| **Multilevel** | A chain: C derives from B, B derives from A | `class C : public B {};` where `B : public A` |
| **Hierarchical** | Several derived classes share **one** base | `class B : public A {}; class C : public A {};` |
| **Multiple** | **One** derived class has **several** bases | `class C : public A, public B {};` |
| **Hybrid** | Any combination of the above (typically hierarchical + multiple → the diamond) | needs `virtual` inheritance |

**"Multiple vs multilevel"** — the single most commonly confused pair:

- **Multiple**: one child, *many parents*, at **one level**. `class Child : public Father, public Mother {}`.
- **Multilevel**: one parent each, but a *chain across generations*. Grandparent → Parent → Child.

C++ supports all five. **Java and C# do not support multiple inheritance of classes** (only of interfaces) — they avoided the ambiguity problem below by forbidding it.

## The diamond problem and virtual inheritance

Hybrid inheritance creates the **diamond (deadly diamond of death)**: `D` inherits from `B` and `C`, both of which inherit from `A`. Without help, `D` contains **two** copies of `A`, and `d.value` is ambiguous.

```cpp
#include <iostream>
struct Animal        { int age = 0; Animal() { std::cout << "Animal ctor\n"; } };
struct Mammal  : Animal {};
struct WingedAnimal : Animal {};
struct Bat : Mammal, WingedAnimal {};

// Bat b;
// b.age = 5;                 // ERROR: ambiguous — Mammal::Animal::age or WingedAnimal::Animal::age?
// b.Mammal::age = 5;         // works, but now there are genuinely TWO ages. Wrong model.
```

**Fix: `virtual` inheritance** — the shared base is stored once and the most-derived class initialises it directly.

```cpp
#include <iostream>
struct Animal        { int age = 0; Animal() { std::cout << "Animal ctor once\n"; } };
struct Mammal        : virtual Animal {};
struct WingedAnimal  : virtual Animal {};
struct Bat : Mammal, WingedAnimal {};

int main() {
    Bat b;
    b.age = 5;          // unambiguous: exactly ONE Animal sub-object
    std::cout << b.age << "\n";
}
```

Costs of virtual inheritance, worth naming: an extra pointer/offset indirection to reach the shared base, a larger object, and the rule that **the most-derived class is responsible for constructing the virtual base** (intermediate classes' calls to `Animal(...)` are ignored). It is the right tool for interface-style diamonds and rarely for anything else.

## Upcasting, downcasting, and object slicing

```cpp
Manager m("Asha", 50000, 10000);
Employee* e = &m;             // UPCAST: implicit, always safe (a Manager is-a Employee)
std::cout << e->salary();     // 60000 — virtual dispatch finds Manager::salary

Manager* back = dynamic_cast<Manager*>(e);   // DOWNCAST: checked at runtime, needs a virtual fn
if (back) { /* really a Manager */ }         // returns nullptr on failure (throws for references)

Employee copy = m;            // ⚠️ OBJECT SLICING: bonus_ is discarded, only the Employee part copied
std::cout << copy.salary();   // 50000 — and it is NOT polymorphic any more
```

**Object slicing** is a C++-specific hazard with no Java equivalent (Java variables are always references). Rule: **manipulate polymorphic objects through pointers or references, never by value.** Pass `const Employee&`, store `std::unique_ptr<Employee>`.

## Limitations / drawbacks of inheritance

Have three ready:

1. **Tight coupling.** The derived class depends on the base's implementation; a change in the base can silently break every subclass (the *fragile base class problem*).
2. **It's static.** The relationship is fixed at compile time. You cannot change an object's base at runtime the way you can swap a composed member.
3. **Deep hierarchies become unreadable.** Following behaviour means walking up five levels. Inherited members you don't need still bloat the object.
4. **Misused for reuse.** Inheriting just to reuse a method creates a false "is-a". The classic example: `class Stack : public std::vector` — now callers can `insert()` in the middle of your stack. Use **composition** (`class Stack { std::vector<T> data_; }`).
5. **Multiple inheritance ambiguity** (the diamond, above).
6. **It can break the Liskov Substitution Principle.** `class Square : public Rectangle` is the canonical failure: `setWidth`/`setHeight` cannot behave correctly for both.

> **The line to say:** *"Prefer composition over inheritance."* Use inheritance only for genuine "is-a" + polymorphism; use composition for "has-a" and for reuse.

---

# Part 5 — Polymorphism

**Polymorphism** (Greek: *many forms*) means one interface serving multiple underlying types — the same call expression producing different behaviour depending on what's behind it.

Two kinds, split by **when the binding decision is made**:

| | Static / Compile-time | Dynamic / Runtime |
|---|---|---|
| Also called | Early binding, static binding | Late binding, dynamic binding |
| Decided | At compile time | At runtime |
| C++ mechanisms | Function **overloading**, **operator overloading**, **templates**, default arguments | **Virtual functions** via base pointers/references |
| Cost | Zero — can be inlined | One vtable indirection; blocks inlining |
| Flexibility | Types must be known when compiling | Works on types that didn't exist when the caller was compiled |

## Static polymorphism — function overloading

**Overloading** = several functions with the **same name** in the **same scope**, distinguished by their **parameter list**. The compiler picks one by *overload resolution* on the argument types.

```cpp
#include <iostream>
#include <string>

class Printer {
public:
    void print(int i)                  { std::cout << "int: "    << i << "\n"; }
    void print(double d)               { std::cout << "double: " << d << "\n"; }
    void print(const std::string& s)   { std::cout << "string: " << s << "\n"; }
    void print(int a, int b)           { std::cout << "two: " << a << "," << b << "\n"; }
};

int main() {
    Printer p;
    p.print(42);            // int
    p.print(3.14);          // double
    p.print("hi");          // string  (const char* → std::string conversion)
    p.print(1, 2);          // two
}
```

Overloads may differ by: **number** of parameters, **types** of parameters, **order** of types, and **const/volatile or ref-qualification of the member function** (`f() const` vs `f()`).

They may **not** differ by **return type alone** — the compiler has no way to choose, since a call can be made without using the return value.

```cpp
int  f(int);
// double f(int);   // ERROR: redefinition, differs only in return type
```

> **Trap — name hiding:** declaring *any* `f` in a derived class hides *all* base `f` overloads, even ones with different signatures. Fix with `using Base::f;` in the derived class.

## Static polymorphism — operator overloading

**Operator overloading** gives operators a custom meaning for your own types. It's syntactic sugar over a function call, and it exists so user-defined types can read as naturally as built-in ones.

```cpp
#include <iostream>

class Vec2 {
    double x_, y_;
public:
    Vec2(double x = 0, double y = 0) : x_(x), y_(y) {}

    Vec2 operator+(const Vec2& r) const { return Vec2(x_ + r.x_, y_ + r.y_); }   // member
    Vec2& operator+=(const Vec2& r) { x_ += r.x_; y_ += r.y_; return *this; }
    bool operator==(const Vec2& r) const { return x_ == r.x_ && y_ == r.y_; }
    double operator[](int i) const { return i == 0 ? x_ : y_; }

    friend std::ostream& operator<<(std::ostream& os, const Vec2& v) {          // non-member
        return os << "(" << v.x_ << ", " << v.y_ << ")";
    }
};

int main() {
    Vec2 a(1, 2), b(3, 4);
    std::cout << a + b << "\n";    // (4, 6)   ← calls a.operator+(b), then operator<<
}
```

Rules to quote:
- **Cannot be overloaded:** `.` `.*` `::` `?:` `sizeof` `typeid` `alignof`, and `#`/`##`.
- You **cannot invent new operators** or change **arity**, **precedence**, or **associativity**.
- At least one operand must be a user-defined type — you can't redefine `int + int`.
- `=`, `[]`, `()`, `->` must be **non-static member functions**.
- Prefer **non-member** for symmetric binary operators (so `2 * v` works as well as `v * 2`) and for `<<` / `>>`.
- **Guideline:** overload only when the meaning is obvious and matches the built-in intuition. `+` on a `Matrix`: good. `+` meaning "launch" on a `Rocket`: a crime.

⚠️ **Java does not support operator overloading at all** (except the built-in `String +`). C# and Python do. Mention this if asked to compare.

## Dynamic polymorphism — virtual functions and overriding

**Overriding** = a derived class supplies its own implementation of a **virtual** function it inherited, with the **same signature**. Which one runs is decided **at runtime**, from the actual object type — not the pointer's static type.

A **virtual function** is a member function declared `virtual` in a base class, intended to be redefined in derived classes and dispatched dynamically.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Animal {
public:
    virtual void speak() const { std::cout << "...\n"; }   // virtual → overridable
    void walk() const { std::cout << "walking\n"; }        // non-virtual → NOT overridable
    virtual ~Animal() = default;                           // essential, see below
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Woof\n"; } // `override` = compiler-checked
};

class Cat : public Animal {
public:
    void speak() const override { std::cout << "Meow\n"; }
};

int main() {
    std::vector<std::unique_ptr<Animal>> zoo;
    zoo.push_back(std::make_unique<Dog>());
    zoo.push_back(std::make_unique<Cat>());
    for (const auto& a : zoo) a->speak();     // Woof, Meow — one call site, many behaviours
}
```

**Always write `override`.** It costs nothing and turns a silent bug (typo'd signature → you created an unrelated new function that never gets called) into a compile error. `final` on a virtual function forbids further overriding; on a class, forbids deriving from it.

## How virtual dispatch actually works (vtable / vptr)

Be able to sketch this — it separates people who memorised from people who understand.

- Every class with at least one virtual function gets a compiler-generated **vtable**: a static array of function pointers, one per virtual function, filled in with that class's final overriders.
- Every *object* of such a class carries a hidden **vptr** pointing to its class's vtable. That's why `sizeof` grows by one pointer.
- `a->speak()` compiles to roughly: *load the vptr from the object → index slot N → call through that pointer.* The slot index is fixed at compile time; the table it indexes into is not.

```cpp
struct NonVirt { int x; };            // sizeof == 4
struct Virt    { int x; virtual ~Virt() = default; };   // sizeof == 16 on x86-64 (vptr + padding)
```

Consequences worth stating: one extra indirection per call, the call generally can't be inlined, and the vptr makes the object non-trivially-copyable in layout terms.

## Virtual destructors — the single most important C++ OOP rule

```cpp
class Base    { public: ~Base()  { std::cout << "~Base\n";  } };        // NOT virtual — bug
class Derived : public Base {
    int* data_ = new int[100];
public: ~Derived() { delete[] data_; std::cout << "~Derived\n"; }
};

Base* p = new Derived();
delete p;                 // ⚠️ UNDEFINED BEHAVIOUR: only ~Base runs. ~Derived never does. Leak.
```

**Rule: if a class has any virtual function, or is ever deleted through a base pointer, its destructor must be `virtual`.** Marking `virtual ~Base() = default;` fixes the above and costs nothing you weren't already paying (the class already has a vptr).

Corollary: if a class is **not** meant to be a polymorphic base, don't give it a virtual destructor — give it a `protected` non-virtual destructor, or mark the class `final`.

## Overloading vs Overriding — the comparison table

| | **Overloading** | **Overriding** |
|---|---|---|
| Meaning | Same name, **different parameter list** | Same name **and** same signature, redefined in a derived class |
| Scope | Same class (or same scope) | Across a base–derived pair |
| Binding | **Compile time** (static / early) | **Runtime** (dynamic / late) |
| Polymorphism | Static | Dynamic |
| Inheritance needed | No | Yes |
| `virtual` needed | No | Yes (in the base) |
| Return type | Must be usable; can differ, but can't be the *only* difference | Must be identical, or a **covariant** pointer/reference |
| Parameters | Must differ | Must be identical |
| Access | Any | Can be widened or narrowed, but narrowing is bad practice |
| C++ marker | — | `override` |

```cpp
// covariant return type — legal and useful for clone()
struct Base    { virtual Base*    clone() const { return new Base(*this); }
                 virtual ~Base() = default; };
struct Derived : Base { Derived* clone() const override { return new Derived(*this); } };
```

## Bonus: static polymorphism via templates (CRTP)

If asked "can you get polymorphism without virtual functions?" — yes, with templates, resolved entirely at compile time and with zero overhead:

```cpp
#include <iostream>
template <typename Derived>
struct Shape {                                   // Curiously Recurring Template Pattern
    void describe() const {
        std::cout << "area = " << static_cast<const Derived*>(this)->area() << "\n";
    }
};
struct Sq : Shape<Sq> {
    double s; explicit Sq(double s) : s(s) {}
    double area() const { return s * s; }
};
// Sq(3).describe();   →  area = 9   — dispatched at compile time, inlinable, no vtable
```

---

# Part 6 — Constructors, Destructors, and Object Lifetime

## Constructors

A **constructor** is a special member function that runs automatically when an object is created, to bring it into a valid state. It has the **same name as the class**, has **no return type** (not even `void`), and **cannot be `virtual`**, `const`, or `static`.

### Types of constructors

```cpp
#include <iostream>
#include <string>
#include <initializer_list>

class Str {
    std::string s_;
public:
    Str() : s_("") { std::cout << "1 default\n"; }                    // 1. Default
    explicit Str(std::string s) : s_(std::move(s)) { std::cout << "2 parameterized\n"; }
    Str(const Str& o) : s_(o.s_) { std::cout << "3 copy\n"; }         // 3. Copy
    Str(Str&& o) noexcept : s_(std::move(o.s_)) { std::cout << "4 move\n"; }  // 4. Move (C++11)
    Str(std::initializer_list<char> il) : s_(il.begin(), il.end()) {  // 5. initializer_list
        std::cout << "5 init-list\n"; }
    Str(int n, char c) : s_(n, c) { std::cout << "6 delegating target\n"; }
    Str(char c) : Str(1, c) { std::cout << "7 delegating\n"; }        // 7. Delegating (C++11)
    ~Str() = default;
};
```

| Kind | Signature | When it runs |
|---|---|---|
| **Default** | `T()` | `T a;` or `T a{};` — no arguments |
| **Parameterized** | `T(args...)` | `T a(x, y);` |
| **Copy** | `T(const T&)` | Initialising from an existing lvalue |
| **Move** (C++11) | `T(T&&) noexcept` | Initialising from a temporary/`std::move`d object |
| **Delegating** (C++11) | `T(...) : T(...)` | One constructor calls another to avoid duplication |
| **Converting** | any one-arg non-`explicit` ctor | Implicit conversion from that argument type |

> **`explicit`:** a single-argument constructor that isn't `explicit` creates a silent implicit conversion. `void f(Str); f("oops");` would compile. Mark single-argument constructors `explicit` unless the conversion is genuinely desirable.

### Member initializer lists — use them

```cpp
class Good {
    const int id_;            // const member: CAN ONLY be set in the init list
    std::string name_;
    Widget& ref_;             // reference member: same
public:
    Good(int id, std::string n, Widget& w)
        : id_(id), name_(std::move(n)), ref_(w) {}   // initialization
};

class Wasteful {
    std::string name_;
public:
    Wasteful(std::string n) { name_ = n; }   // default-CONSTRUCTS name_, THEN assigns. Two steps.
};
```

Initializer lists are required for `const` members, reference members, base classes, and members without a default constructor — and are more efficient for everything else. **Members are initialised in declaration order**, not in the order you list them; reordering silently breaks dependent initialisation (compile with `-Wall` to catch it).

## The copy constructor (and deep vs shallow copy)

A **copy constructor** creates a new object as a copy of an existing one. Signature: `T(const T& other)`.

It is called when you: initialise from another object (`T b = a;` / `T b(a);`), pass by value, or return by value (though copy elision usually removes that).

**Why write your own?** Because the compiler's default does a **member-wise (shallow) copy**. If your class owns a raw pointer, both objects end up pointing at the same buffer → double `delete` → crash.

```cpp
#include <algorithm>
#include <cstddef>

class Buffer {
    int* data_;
    std::size_t n_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]()), n_(n) {}

    Buffer(const Buffer& o) : data_(new int[o.n_]), n_(o.n_) {      // DEEP copy
        std::copy(o.data_, o.data_ + o.n_, data_);
    }
    Buffer& operator=(const Buffer& o) {                            // copy assignment
        if (this != &o) {                                           // self-assignment check
            int* tmp = new int[o.n_];                               // allocate BEFORE freeing
            std::copy(o.data_, o.data_ + o.n_, tmp);
            delete[] data_;
            data_ = tmp; n_ = o.n_;
        }
        return *this;
    }
    ~Buffer() { delete[] data_; }
};
```

| Shallow copy | Deep copy |
|---|---|
| Copies pointer values | Copies the pointed-to data |
| Both objects share the resource | Each object owns its own |
| Default compiler-generated behaviour | You must write it (or use an owning member) |
| Double-free / dangling risk | Safe, but costs an allocation |

## The Rule of Three / Five / Zero

**Rule of Three (C++98):** if you need to write *any one* of the **destructor**, **copy constructor**, or **copy assignment operator**, you almost certainly need all three — because the need signals that the class manages a resource.

**Rule of Five (C++11):** add the **move constructor** and **move assignment operator**, or you lose move optimisations.

```cpp
class R {
public:
    ~R();                              // 1 destructor
    R(const R&);                       // 2 copy constructor
    R& operator=(const R&);            // 3 copy assignment
    R(R&&) noexcept;                   // 4 move constructor
    R& operator=(R&&) noexcept;        // 5 move assignment
};
```

**Rule of Zero (what to actually do):** design classes so they need *none* of these. Hold resources in types that already manage themselves — `std::string`, `std::vector`, `std::unique_ptr`, `std::shared_ptr` — and the compiler-generated versions are correct and optimal.

```cpp
class Modern {
    std::vector<int> data_;                  // owns its memory, copies deeply, moves cheaply
    std::unique_ptr<Widget> w_;              // move-only, frees automatically
    // no destructor, no copy/move ctor, nothing to get wrong
};
```

You can also `= default` or `= delete` any of them:
```cpp
class NonCopyable {
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;             // copying is a compile error
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

## Destructors

A **destructor** runs automatically when an object's lifetime ends, to release whatever the object acquired. Named `~ClassName()`, takes no parameters, returns nothing, **cannot be overloaded** (there is exactly one per class), and **can and often should be `virtual`**.

It runs when: a stack object leaves scope, a `delete` is executed, a temporary expires at the end of its full expression, a containing object is destroyed, or the program exits (for statics).

```cpp
class File {
    std::FILE* f_;
public:
    explicit File(const char* path) : f_(std::fopen(path, "r")) {
        if (!f_) throw std::runtime_error("open failed");
    }
    ~File() { if (f_) std::fclose(f_); }      // guaranteed, even if an exception unwinds
};
```

**Destructors should not throw.** They are implicitly `noexcept` since C++11; throwing from one during stack unwinding calls `std::terminate`.

## RAII — the idea that replaces `finalize`, `finally`, and GC

**Resource Acquisition Is Initialization**: bind a resource's lifetime to an object's lifetime. Acquire in the constructor, release in the destructor. The compiler then guarantees release — on normal return, on `break`, on early `return`, and on exception unwinding.

```cpp
{
    std::lock_guard<std::mutex> lock(m);   // acquires
    doWork();                              // even if this throws...
}                                          // ...the mutex is released here. Always.
```

This single idea is why C++ needs neither `finally` nor a garbage collector for deterministic resources.

## Memory management: "finalize" and "Garbage Collection"

⚠️ **Java-isms — this is where you score points.**

**"What is the use of `finalize`?"** — C++ has **no `finalize`**. Java's `finalize()` was a method the GC *might* call before reclaiming an object, at an *unspecified* time, or **never**. It was unreliable enough that Java deprecated it in Java 9 and removed it in Java 18. **The C++ answer is the destructor**, which is *deterministic*: it runs at a known point, in a known order, guaranteed.

**"What is Garbage Collection?"** — GC is automatic reclamation of unreachable heap memory, used by Java, C#, Go, Python (refcount + cycle collector). **C++ has no garbage collector by design.** Instead:

| Approach | How memory is freed | Notes |
|---|---|---|
| **Automatic (stack) storage** | At scope exit | The default; free, instant |
| **`std::unique_ptr<T>`** | When the unique owner is destroyed | Zero overhead vs a raw pointer; move-only |
| **`std::shared_ptr<T>`** | When the last owner is destroyed (reference counting) | Has overhead; watch for reference **cycles** |
| **`std::weak_ptr<T>`** | Non-owning; breaks `shared_ptr` cycles | `.lock()` to use |
| **`new` / `delete`** | Manually, by you | Avoid in modern code |

```cpp
{
    auto w = std::make_unique<Widget>();   // allocated
    // ... use w ...
}                                          // freed here. Deterministically. No GC pause.
```

**Why C++ chose no GC:** determinism (destructors fire immediately — critical for file handles, sockets, locks, not just memory), no unpredictable pause times (real-time and game code), and no runtime overhead you didn't ask for. The trade: you must think about ownership. Smart pointers make that cheap.

**Also know:** `delete` vs `delete[]` (mismatching them is UB), and that `new` calls the constructor while `malloc` does not.

## "What is a final variable?"

⚠️ **Java-ism.** Java's `final` variable is one that can be assigned once. C++ has **`const`** and **`constexpr`**:

```cpp
int x = 0;

const int     a = 5;      // runtime constant — cannot be modified after initialisation
constexpr int b = 5;      // compile-time constant — usable as an array size or template argument

const int* p = &x;        // pointer to const int   → cannot change *p, can change p
int* const q = &x;        // const pointer to int   → can change *q, cannot change q
const int* const r = &x;  // both

struct S {
    void f() const;       // const member function  → promises not to modify *this
};
```

C++ *does* have the keyword **`final`**, but it means something different: it applies to **classes** ("cannot be derived from") and **virtual functions** ("cannot be overridden further"), not to variables.

```cpp
class Sealed final { };                       // nothing may inherit from Sealed
struct B { virtual void f(); };
struct D : B { void f() final; };             // no further override of f() below D
```

---

# Part 7 — Exception Handling

## What is an exception?

An **exception** is an object thrown to signal that a function could not fulfil its contract — an abnormal condition that disrupts normal flow, detected at one point and handled at another, possibly far up the call stack.

Its purpose: separate **error detection** from **error handling**, and make errors impossible to ignore silently (unlike a return code nobody checks).

## Error vs Exception

| | **Error** | **Exception** |
|---|---|---|
| Nature | A serious problem, usually unrecoverable | An abnormal but recoverable condition |
| Examples | Stack overflow, out-of-memory, hardware fault, a compile-time type error | File not found, invalid input, index out of range, network timeout |
| Should you catch it? | Generally no — you can't meaningfully continue | Yes, that's the point |
| In C++ | Not a language category; ~ `std::bad_alloc`, signals, UB, or a hard crash | `std::exception` hierarchy |
| In Java | `java.lang.Error` (a distinct class) | `java.lang.Exception` |

Note that in C++ the distinction is conceptual rather than enforced by the type system; Java bakes it into the class hierarchy. Also: C++ has **no checked exceptions** — there is no `throws` clause you must declare or handle. (`throw()` / dynamic exception specifications were removed in C++17; only `noexcept` remains.)

## try / catch / throw

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>

double divide(int a, int b) {
    if (b == 0) throw std::invalid_argument("division by zero");   // THROW: raise it
    return static_cast<double>(a) / b;
}

int main() {
    try {                                                 // TRY: the guarded region
        std::cout << divide(10, 2) << "\n";
        std::cout << divide(1, 0) << "\n";                // throws; rest of try is skipped
    }
    catch (const std::invalid_argument& e) {              // CATCH: most-derived FIRST
        std::cout << "invalid argument: " << e.what() << "\n";
    }
    catch (const std::exception& e) {                     // then the base
        std::cout << "std exception: " << e.what() << "\n";
    }
    catch (...) {                                         // catch-all, must be LAST
        std::cout << "unknown exception\n";
        // throw;                                         // rethrow, preserving the original
    }
}
```

Rules that get asked:
- **Throw by value, catch by `const` reference.** Catching by value slices a derived exception into its base.
- **Catch clauses are tried in order**, so put derived types before base types — otherwise the base catch swallows everything and the compiler warns about an unreachable handler.
- If no handler matches anywhere up the stack, `std::terminate` is called.
- **Stack unwinding**: as the exception propagates, every automatic object between the throw and the handler is destroyed properly. This is what makes RAII work.

The standard hierarchy: `std::exception` → `logic_error` (`invalid_argument`, `out_of_range`, `domain_error`, `length_error`) and `runtime_error` (`overflow_error`, `underflow_error`, `range_error`, `system_error`), plus `bad_alloc`, `bad_cast`, `bad_optional_access`.

## "What is a finally block?"

⚠️ **Java-ism. C++ has no `finally`.** Java's `finally` runs whether or not an exception occurred, and is used for cleanup (closing files, releasing locks).

**C++ doesn't need one, because destructors already do that job.** This is RAII again, and the answer an interviewer wants to hear:

```cpp
// Java                                  |  // C++
// FileWriter f = null;                  |  {
// try {                                 |      std::ofstream f("out.txt");   // acquires
//     f = new FileWriter("out.txt");    |      f << data;                    // may throw
//     f.write(data);                    |  }   // ~ofstream ALWAYS runs — closes the file
// } finally {                           |
//     if (f != null) f.close();         |  // No finally. No leak possible. Less code.
// }                                     |
```

C++ cleanup is tied to the *object*, so it's written once in the class and cannot be forgotten at a call site. Java 7 later added try-with-resources, which is an acknowledgement that RAII was the better idea.

If you truly need ad-hoc cleanup, write a scope guard: `std::unique_ptr` with a custom deleter, or a tiny struct whose destructor runs a lambda.

## Exception safety guarantees

Worth naming if the conversation goes deep:

1. **No-throw (`noexcept`)** — never throws. Required of destructors, swap, and move operations.
2. **Strong** — if it throws, state is unchanged, as if the call never happened (the *copy-and-swap* idiom gives you this).
3. **Basic** — if it throws, no leaks and all invariants hold, but the state may have changed.
4. **None** — anything goes. Unacceptable in library code.

---

# Part 8 — Rapid-Fire: All 50 Questions, Short Answers

Answers sized to say out loud in 15–40 seconds. Numbering follows the article.

### Basics

**1. Difference between OOP and SOP?**
SOP (structured/procedural) organises code around functions acting on separate, exposed data; it decomposes top-down. OOP organises code around objects that own their data and expose only valid operations; it composes bottom-up. OOP adds encapsulation, inheritance and polymorphism, so it scales better when you keep adding new *types*; procedural code is simpler and better when you keep adding new *operations* over a fixed set of types.

**2. What is OOPs?**
A programming paradigm that models a system as interacting objects — bundles of data (state) and functions (behaviour) — built from classes, with encapsulation, abstraction, inheritance and polymorphism as its defining features.

**3. Why use OOPs?**
Reusability (inheritance, composition, templates), maintainability (changes stay behind an interface), extensibility (new behaviour = new class, not edits everywhere), data security via encapsulation, and a design that mirrors the problem domain.

**4. Main features of OOPs?**
Encapsulation, abstraction, inheritance, polymorphism. (Plus message passing, on five-pillar lists.)

### Classes and objects

**5. What is an object?**
A runtime instance of a class — a region of memory with its own state, plus identity and behaviour. `Car c("Swift", 60);`

**6. What is a class?**
A user-defined type; a blueprint specifying what data an instance holds and what operations it supports. It's a compile-time construct and occupies no runtime memory by itself.

**7. Difference between a class and a structure?**
In C++, only the default access: `struct` defaults to `public` (members and inheritance), `class` to `private`. Both support constructors, inheritance, virtual functions and templates. By convention, `struct` for passive data with no invariant, `class` when you must protect state. (In C, a struct has no member functions at all; in C#, struct is a value type — a much bigger difference.)

**8. Can you call the base class method without creating an instance?**
Yes if it's `static` (`Base::method()`), or through a derived object (which contains a base sub-object) — `d.Base::method()`. No for a non-static member function with no object at all, since it needs a `this`.

**9. Difference between a class and an object?**
A class is the definition/type — one per concept, compile time, no memory. An object is an instance — many per class, runtime, with its own memory and state. Cookie cutter vs cookie.

### Inheritance

**10. What is inheritance?**
A mechanism where a derived class acquires the members of a base class, modelling "is-a" and enabling reuse plus runtime polymorphism. `class Dog : public Animal {};`

**11. Types of inheritance?**
Single, multilevel, hierarchical, multiple, hybrid. C++ supports all five. (C++ also has three *access modes* — public, protected, private inheritance — which is a separate axis.)

**12. Multiple vs multilevel inheritance?**
Multiple: one derived class, several direct bases, all at one level (`class C : public A, public B`). Multilevel: a chain across generations — A → B → C, each with a single parent. Multiple can produce the diamond problem; multilevel can't.

**13. What is hybrid inheritance?**
A combination of two or more inheritance types in one hierarchy — typically hierarchical plus multiple, producing the diamond, which C++ resolves with `virtual` base classes.

**14. What is hierarchical inheritance?**
One base class with several independent derived classes — `Shape` → `Circle`, `Square`, `Triangle`. Shared behaviour lives once in the base.

**15. Limitations of inheritance?**
Tight coupling to the base (fragile base class); the relationship is fixed at compile time; deep hierarchies hurt readability and object size; it's frequently abused for code reuse where composition fits better; multiple inheritance introduces ambiguity; and it can break Liskov substitution (`Square : Rectangle`). Prefer composition over inheritance.

**16. What is a superclass?**
The base/parent class — the generalisation being inherited from. It holds members common to all its subclasses.

**17. What is a subclass?**
The derived/child class — the specialisation. It has everything the superclass has, plus its own additions and overrides.

### Polymorphism

**18. What is polymorphism?**
"Many forms" — one interface with multiple behaviours, so the same call expression does different things depending on the actual type. Static (compile-time) and dynamic (runtime).

**19. What is static polymorphism?**
Resolved at compile time (early binding). In C++: function overloading, operator overloading, templates/CRTP, default arguments. Zero runtime cost and inlinable, but the types must be known when compiling.

**20. What is dynamic polymorphism?**
Resolved at runtime (late binding) through `virtual` functions called via base pointers or references. The vptr in the object selects the actual override. Costs one indirection; buys the ability to work with types the caller was never compiled against.

**21. What is method overloading?**
Several functions sharing a name in one scope, differing in number, type, or order of parameters (or const/ref-qualification for members). Resolved at compile time. **Cannot differ by return type alone.**

**22. What is method overriding?**
A derived class redefining an inherited `virtual` function with the identical signature, so calls through a base pointer/reference dispatch to the derived version at runtime. Mark it `override`.

**23. What is operator overloading?**
Giving an operator custom meaning for a user-defined type — it's a function call in disguise: `a + b` → `a.operator+(b)`. You cannot overload `.` `.*` `::` `?:` `sizeof` `typeid`, invent new operators, or change arity/precedence/associativity. Java doesn't support it at all.

**24. Overloading vs overriding?**
Overloading: same name, *different* parameters, same scope, no inheritance needed, bound at *compile* time. Overriding: same name *and* signature, across base→derived, requires `virtual`, bound at *runtime*. Overloading is static polymorphism, overriding is dynamic.

### Encapsulation

**25. What is encapsulation?**
Bundling data with the functions that operate on it into one unit, and hiding the data behind a controlled interface, so invariants are enforced in one place and the implementation can change without breaking callers.

**26. What are access specifiers?**
Keywords controlling which code may access a member: `public`, `protected`, `private`. They implement data hiding. C++ also applies them to inheritance (`class D : private B`), which Java/C# don't.

**27. public vs private vs protected?**
`public` — accessible anywhere; the interface. `protected` — accessible within the class and its derived classes; for members subclasses need but outsiders must not touch. `private` — accessible only within the class and its `friend`s; the default for `class`, and where state should live.

### Abstraction

**28. What is data abstraction?**
Exposing only the essential features of an entity and hiding implementation details — describing *what* an object does, not *how*. You drive a car by its interface; the engine is hidden.

**29. How to achieve data abstraction?**
Abstract classes with pure virtual functions; access specifiers (private data, public interface); header/implementation separation; the Pimpl idiom; and templates/concepts for compile-time abstraction.

**30. What is an abstract class?**
A class with at least one pure virtual function (`virtual f() = 0;`). It defines an interface (and optionally shared implementation) for its derived classes and cannot be instantiated. Give it a virtual destructor.

**31. Can you create an instance of an abstract class?**
No — the compiler rejects it, since pure virtual functions have no implementation to dispatch to. You can declare pointers and references to it, which is how polymorphism works. Its constructor still runs as part of constructing a derived object.

**32. What is an interface?**
A contract: a set of function signatures a type promises to provide, with no implementation. C++ has no `interface` keyword — you write a class where every function is pure virtual and there's no data. Since C++ supports multiple inheritance, one class can implement many. C++20 concepts are the compile-time equivalent.

**33. Data abstraction vs encapsulation?**
Abstraction is a *design-level* concern — deciding what to expose, hiding complexity, solved with abstract classes and interfaces. Encapsulation is an *implementation-level* concern — enforcing that hiding, solved with access specifiers. Abstraction hides complexity; encapsulation hides data. Abstraction defines the interface; encapsulation protects it.

### Methods and functions

**34. What are virtual functions?**
Member functions declared `virtual` in a base class so that a call through a base pointer/reference dispatches to the derived override at runtime, via the object's vptr and the class's vtable. They're the mechanism of dynamic polymorphism.

**35. What are pure virtual functions?**
Virtual functions declared `= 0` with no required implementation, which derived classes **must** override. A class containing one is abstract. They may still have a body that derived classes call explicitly, and a pure virtual *destructor* must have one.

**36. What is a constructor?**
A special member function with the class's name and no return type, called automatically on object creation to establish a valid initial state. Can be overloaded; cannot be `virtual`, `const`, or `static`.

**37. What is a destructor?**
`~ClassName()` — called automatically when an object's lifetime ends, to release its resources. Exactly one per class, no parameters, no return type, cannot be overloaded, should not throw, and **must be `virtual` in any polymorphic base class**.

**38. Types of constructors?**
Default, parameterized, copy, move (C++11), delegating (C++11), and converting (any non-`explicit` one-arg constructor). Java has no copy or move constructors in this sense.

**39. What is a copy constructor?**
`T(const T& other)` — creates a new object as a copy of an existing one. Called on copy-initialisation, pass-by-value, and return-by-value. The compiler-generated one is a shallow member-wise copy, so any class owning a raw resource needs a deep-copying version — and, by the Rule of Three/Five, a destructor and copy-assignment too. Best answer: use `std::vector`/`unique_ptr` and follow the Rule of Zero.

**40. Use of `finalize`?**
⚠️ Java concept; **C++ has none**. Java's `finalize()` was a pre-GC hook that ran at an unpredictable time, or never — deprecated in Java 9, removed in 18. C++'s equivalent is the **destructor**, which is deterministic: it runs at a known point in a known order, guaranteed, including during exception unwinding.

**41. What is Garbage Collection?**
Automatic reclamation of unreachable heap memory (Java, C#, Go, Python). **C++ has none by design** — it uses scope-based automatic storage and RAII smart pointers (`unique_ptr`, `shared_ptr`, `weak_ptr`) instead. The trade is determinism and zero runtime overhead in exchange for thinking about ownership.

**42. Class vs method?**
A class is a *type* — a blueprint defining state and behaviour. A method (member function) is *one behaviour* that belongs to a class and operates on an object's state via `this`. A class contains methods; a method cannot contain a class instance definition as its identity.

**43. Abstract class vs interface?**
An abstract class may hold data and provide partial implementations, models "is-a", and is used to share code up a hierarchy. An interface (pure abstract class in C++) has no data and no implementations, models "can-do", and is used to declare a contract. In C++ both are just classes; the distinction is a design convention, and multiple inheritance makes multiple interfaces natural. In Java they're separate language constructs.

**44. What is a final variable?**
⚠️ Java concept. C++ uses **`const`** (cannot be modified after initialisation) and **`constexpr`** (known at compile time). C++'s own `final` keyword is unrelated: it seals a **class** against derivation or a **virtual function** against further overriding.

### Exception handling

**45. What is an exception?**
An object thrown to signal that a function couldn't meet its contract — an abnormal condition that disrupts normal flow and propagates up the stack until a matching handler catches it. It makes errors impossible to ignore silently.

**46. What is exception handling?**
The mechanism for detecting and responding to exceptions — `throw` to raise, `try` to guard a region, `catch` to handle — separating error detection from error handling and keeping the happy path readable. In C++ it also triggers **stack unwinding**, which destroys every automatic object along the way (the basis of RAII).

**47. Error vs exception?**
An error is typically serious and unrecoverable (out of memory, stack overflow, hardware fault) and shouldn't be caught; an exception is an abnormal but recoverable condition (file not found, bad input) that you're expected to handle. Java enforces the split with `Error` vs `Exception` classes; in C++ it's conceptual, though `std::logic_error` (a bug — fix the code) vs `std::runtime_error` (a condition — handle it) is the analogous distinction.

**48. What is a try/catch block?**
`try` encloses code that might throw; `catch` immediately follows with handlers for specific exception types. On a throw, the rest of the `try` is skipped and the first matching `catch` runs. Order handlers most-derived first, catch by `const` reference (catching by value slices), and `catch (...)` last as a catch-all.

**49. What is a `finally` block?**
⚠️ Java concept — code that runs whether or not an exception occurred, used for cleanup. **C++ has no `finally` and doesn't need one**: destructors of automatic objects run during stack unwinding, so RAII (`lock_guard`, `unique_ptr`, `ofstream`) does the cleanup automatically, written once in the class instead of repeated at every call site.

### Wrap-up

**50. Limitations of OOPs?**
Steeper learning curve and more upfront design; more code and boilerplate for simple problems; runtime cost where dynamic dispatch and indirection are used (vtable lookups, cache-unfriendly pointer chasing); over-engineering is easy (deep hierarchies, classes for everything); inheritance creates tight coupling and fragile base classes; and it's a poor fit for some domains — data-oriented, numeric, and highly concurrent code often prefer procedural, data-oriented or functional designs. Finally, the Expression Problem: OOP makes adding new *types* easy and adding new *operations* hard.

---

# Part 9 — Cheat Sheet, Traps, and Practice

## Traps interviewers set

| The trap | The answer |
|---|---|
| "Can a constructor be virtual?" | **No.** Virtual dispatch needs a vptr, which the constructor is what sets up. (Achieve the effect with a virtual `clone()` — the *virtual constructor idiom*.) |
| "Can a destructor be virtual?" | **Yes — and it must be** in any class deleted through a base pointer. Otherwise: undefined behaviour and a leak. |
| "Can a static function be virtual?" | **No.** `static` means no `this`; `virtual` dispatch needs `this`. |
| "Can you overload on return type alone?" | **No.** The compiler couldn't choose when the value is discarded. |
| "Can you override a non-virtual function?" | **No — you'd be *hiding* it.** Calls through a base pointer still run the base version. |
| "What happens if you call a virtual function in a constructor?" | The **base** version runs — the derived part doesn't exist yet. (Java calls the override, on a half-built object.) |
| "Can a class be both abstract and have a constructor?" | **Yes.** It runs as part of constructing derived objects. |
| "Can a friend function be inherited / virtual?" | **No** to both — friendship isn't inherited and isn't transitive. |
| "What's the size of an empty class?" | **1 byte** — so that distinct objects have distinct addresses. With a virtual function, typically 8 (the vptr). |
| "`Derived d; Base b = d;` — what happens?" | **Object slicing.** Only the `Base` sub-object is copied; polymorphism is lost. |
| "Does C++ support multiple inheritance?" | **Yes**, unlike Java/C#, with `virtual` base classes to resolve the diamond. |
| "Private members are inherited?" | **Yes, they exist in the derived object** — but they're **not accessible** from it. Different things. |
| "Can you have a pure virtual function with a body?" | **Yes**, and a pure virtual *destructor* is required to have one. |

## The 60-second recap

```
ENCAPSULATION   private data + public methods       → protects invariants
ABSTRACTION     pure virtual / abstract base        → hides complexity
INHERITANCE     class D : public B                  → "is-a", enables polymorphism
POLYMORPHISM    overload (compile) / virtual (run)  → one interface, many behaviours

MUST-SAY C++ SPECIFICS
  virtual destructor in every polymorphic base
  Rule of Zero → prefer vector/string/unique_ptr; Rule of Five if you must
  RAII replaces finally and GC — destructors are deterministic
  pass polymorphic objects by reference/pointer, never by value (slicing)
  mark overrides `override`, single-arg ctors `explicit`, non-mutating methods `const`
  composition over inheritance
```

## SOLID — say these if design comes up

- **S**ingle Responsibility — a class should have one reason to change.
- **O**pen/Closed — open for extension, closed for modification (add a subclass, don't edit the base).
- **L**iskov Substitution — a derived object must be usable anywhere its base is, without surprises.
- **I**nterface Segregation — many small interfaces beat one fat one; don't force clients to depend on methods they don't use.
- **D**ependency Inversion — depend on abstractions, not concretions (take a `Logger&`, not a `FileLogger&`).

## Practice: write these from scratch

Each is a standard whiteboard ask. Time yourself; 15–20 minutes each.

1. **`Shape` hierarchy** — abstract base with `area()` and `perimeter()`, `Circle`/`Rectangle`/`Triangle`, a `vector<unique_ptr<Shape>>`, total area. *Tests: abstract classes, virtual dispatch, virtual destructor, smart pointers.*
2. **A `String` class** — own a `char*`, implement the full Rule of Five plus `operator+`, `operator[]`, `operator<<`. *Tests: deep copy, move semantics, operator overloading, self-assignment.*
3. **`Stack<T>`** — implement with composition over `std::vector`, then explain why not `class Stack : public std::vector`. *Tests: composition vs inheritance, templates.*
4. **Bank account hierarchy** — `Account` base, `Savings`/`Current` with different `withdraw()` rules and interest. Throw on overdraft. *Tests: encapsulation, virtual overrides, exceptions.*
5. **The diamond** — build `Bat : Mammal, WingedAnimal : Animal`, show the ambiguity error, then fix it with `virtual` inheritance and explain who constructs `Animal`.
6. **A `unique_ptr`** — write a minimal `SmartPtr<T>` with constructor, destructor, deleted copy, move constructor and move assignment, `operator*`, `operator->`. *Tests: RAII, move semantics, ownership.*
7. **Observer pattern** — `Subject` holds `vector<Observer*>`, `notify()` calls each `observer->update()`. *Tests: interfaces, dynamic polymorphism, and one design pattern to name-drop.*

## Compile everything you write

```bash
g++ -std=c++17 -Wall -Wextra -pedantic file.cpp -o prog && ./prog
g++ -std=c++17 -fsanitize=address,undefined -g file.cpp && ./a.out   # catches leaks and UB
```

`-Wall -Wextra` will catch the two mistakes you're most likely to make under interview pressure: member initialisation out of declaration order, and a non-virtual destructor in a polymorphic base.

---

**Source:** [OOPs Interview Questions — Edureka on Medium](https://medium.com/edureka/oops-interview-questions-621fc922cdf4) (question list; answers rewritten for C++).
