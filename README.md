# 🏦 Project 3.2 — Bank System (2nd Extension)

> The Bank System grows into a multi-user platform — login, role-based bitwise permissions, and a full User Management module layered cleanly on top of everything that was already working.

---

## 🚀 Project Overview

This is the **second extension** of the Bank Client Management System built in C++.

The core client operations were already working from Projects 3 and 3.1.

This extension adds a completely new dimension:

**Who is allowed to do what?**

A login screen comes first — every session starts with a username and password.

Once logged in, a user can only access the operations their permissions allow.

A Manage Users menu lets admins create new users, define what each one can do, and remove or update existing ones.

No existing feature was broken.

The new layer plugged in cleanly — because the base was built correctly.

---

## 🏗️ Architecture Design

```
main()
 └── Login()          ← loops until valid credentials
      └── ShowMainMenue()
           ├── [1] ShowAllClientsScreen()      ← CheckAccess(pListClients)
           ├── [2] ShowAddNewClientsScreen()   ← CheckAccess(pAddNewClient)
           ├── [3] ShowDeleteClientScreen()    ← CheckAccess(pDeleteClient)
           ├── [4] ShowUpdateClientScreen()    ← CheckAccess(pUpdateClient)
           ├── [5] ShowFindClientScreen()      ← CheckAccess(pFindClient)
           ├── [6] ShowTransactionsMenue()     ← CheckAccess(pTransactions)
           │    ├── [1] Deposit
           │    ├── [2] Withdraw
           │    ├── [3] Total Balances
           │    └── [4] Back to Main
           ├── [7] ShowManageUsersMenue()      ← CheckAccess(pManageUsers)
           │    ├── [1] List Users
           │    ├── [2] Add New User
           │    ├── [3] Delete User      ← blocked for "Admin"
           │    ├── [4] Update User      ← blocked for "Admin"
           │    ├── [5] Find User
           │    └── [6] Back to Main
           └── [8] Logout → Login()
```

---

## ⚙️ Core Functionalities

### Inherited from Projects 3 and 3.1

| Operation | Description |
|---|---|
| Client CRUD | Add, delete, update, find, and list clients |
| Transactions | Deposit, withdraw (with balance validation), total balances |
| File persistence | Text file storage with custom separator format |

### New in This Extension

| Feature | Description |
|---|---|
| **Login screen** | Username + password authentication on startup |
| **Session user** | Global `CurrentUser` holds the logged-in user's permissions |
| **Permission check** | Every main menu option validates access before proceeding |
| **User Management** | Full CRUD for users — list, add, delete, update, find |
| **Logout** | Returns to the login screen without closing the program |
| **Admin protection** | Admin account cannot be deleted or updated |

---

## 🧠 Design Decisions Worth Noting

### Bitwise permission system

Each permission is a power of 2:

```cpp
enum enMainMenuePermissions {
    eAll          = -1,  // all bits set — passes any check
    pListClients  =  1,  // bit 0
    pAddNewClient =  2,  // bit 1
    pDeleteClient =  4,  // bit 2
    pUpdateClient =  8,  // bit 3
    pFindClient   = 16,  // bit 4
    pTransactions = 32,  // bit 5
    pManageUsers  = 64   // bit 6
};
```

A user's permission is the SUM of their allowed bits.

For example: permissions = 17 = 16 + 1 = Find Client + List Clients only.

Checking access is a single bitwise AND:

```cpp
bool CheckAccessPermission(enMainMenuePermissions Permission)
{
    if (CurrentUser.Permissions == enMainMenuePermissions::eAll)
        return true;

    return ((Permission & CurrentUser.Permissions) == Permission);
}
```

If the user's bit for that permission is set, access is granted. If not, access is denied.

Adding a new permission in the future requires only one new power of 2 — nothing else changes.

---

### eAll = -1 — all bits set

`-1` in two's complement binary is all 1s.

A bitwise AND against all 1s always returns the original value.

So any permission check against `eAll` passes automatically.

This means full-access users are handled without any special casing — the check itself handles them correctly.

---

### Permission check before screen, not inside it

Every protected menu option checks permissions at entry:

```cpp
void ShowAllClientsScreen()
{
    if (!CheckAccessPermission(enMainMenuePermissions::pListClients))
    {
        ShowAccessDeniedMessage();
        return;
    }
    // ... show the screen
}
```

If denied: show message and return.

The denied screen is never shown. No partial rendering. Clean gate.

---

### Admin account is protected at the function level

```cpp
void DeleteUserByUsername(string Username, vector <stUser>& vUsers)
{
    if (Username == "Admin")
    {
        cout << "\n\nYou cannot Delete This User.\n";
    }
    else if (FindUserByUsername(...))
    { ...
```

Even a user with full Manage Users permission cannot delete the Admin account.

The protection is inside the function — not at the menu level — so it holds regardless of how the function is called.

---

### Function overloading for ConvertRecordToLine

```cpp
string ConvertRecordToLine(stClient& BankClientData, string Seperator = "#//#")
string ConvertRecordToLine(stUser& BankUserData, string Seperator = "#//#")
```

Same function name. Two different structs. The compiler picks the right one.

No `ConvertClientToLine` vs `ConvertUserToLine` naming needed.

---

### Logout = call Login() again

```cpp
case enMainMenueOptions::Logout:
    Login();
    break;
```

Logout is not a close.

It sends the user back to the login screen. The program keeps running.

A new session starts from scratch — with a new `CurrentUser`.

---

## 💾 Data Persistence

### Clients.txt
```
AccountNumber#//#PinCode#//#Name#//#Phone#//#Balance
```

### Users.txt
```
Username#//#Password#//#Permissions
Admin#//#1234#//#-1
User3#//#3333#//#110
User5#//#5555#//#17
```

The permissions number is stored as a plain integer. On load, it is parsed back and used directly in the bitwise checks.

---

## 🛠️ Tech Stack

| | |
|---|---|
| **Language** | C++ |
| **IDE** | Visual Studio |
| **Type** | Console Application |
| **Paradigm** | Structured Programming — Divide & Conquer |
| **Storage** | Text File (File I/O) |
| **STL Used** | `vector`, `string`, `fstream`, `iomanip` |

---

## 📦 Bank Project Series

Each version lives in its **own dedicated repository** with its own full README.

| Version | Repo | Key Additions |
|---|---|---|
| **Bank 1** | *(separate repo)* | Core CRUD, file persistence |
| **Bank 1 — 1st Extension** | *(separate repo)* | Transactions Menu |
| **Bank 1 — 2nd Extension** *(you are here)* | ← this repo | Login, role-based permissions, User Management |

---

## 🏁 Milestone

- ✅ First login system and session management
- ✅ First role-based permission implementation in the series
- ✅ Part of **Course 8 — Algorithms & Problem Solving – Level 4**

---

## 🙏 Gratitude

Thank you to:

**[Programming Advices Platform](https://programmingadvices.com)**

**[Dr. Mohammed Abu-Hadhoud](https://programmingadvices.com)**

---

## 🔥 Final Thought

Clean code is not about the first version.

It is about every version that comes after.

This extension did not require rewriting anything that already worked.

That is the real outcome of building correctly from the start.
