# TeamTaskManager

**System do zarządzania zadaniami w zespołach projektowych z rozbudowaną walidacją, uwierzytelnianiem i śledzeniem postępów.**

---

## Spis treści

1. [Opis projektu](#opis-projektu)  
2. [Funkcjonalności](#funkcjonalności)  
3. [Technologie](#technologie)  
4. [Architektura i struktura projektu](#architektura-i-struktura-projektu)  
   1. [Struktura katalogów](#struktura-katalogów)  
   2. [Modele danych](#modele-danych)  
   3. [Warstwa dostępu do danych (EF Core)](#warstwa-dostępu-do-danych-ef-core)  
   4. [Serwisy biznesowe](#serwisy-biznesowe)  
   5. [Formularze WinForms (UI)](#formularze-winforms-ui)  
   6. [Inicjalizacja i uruchomienie aplikacji](#inicjalizacja-i-uruchomienie-aplikacji)  
5. [Instalacja i konfiguracja](#instalacja-i-konfiguracja)  
   1. [Wymagania wstępne](#wymagania-wstępne)  
   2. [Konfiguracja bazy danych](#konfiguracja-bazy-danych)  
   3. [Budowanie i uruchamianie projektu](#budowanie-i-uruchamianie-projektu)  
6. [Instrukcja obsługi aplikacji](#instrukcja-obsługi-aplikacji)  
   1. [Rejestracja użytkownika](#rejestracja-użytkownika)  
   2. [Logowanie](#logowanie)  
   3. [Panel główny (MainForm)](#panel-główny-mainform)  
   4. [Zarządzanie użytkownikami (UserForm)](#zarządzanie-użytkownikami-userform)  
   5. [Zarządzanie zespołami (TeamForm)](#zarządzanie-zespołami-teamform)  
   6. [Zarządzanie zadaniami (TaskForm)](#zarządzanie-zadaniami-taskform)  
7. [Walidacja i przypadki brzegowe](#walidacja-i-przypadki-brzegowe)  
   1. [Walidacja po stronie UI](#walidacja-po-stronie-ui)  
   2. [Walidacja w serwisach](#walidacja-w-serwisach)  
   3. [Przypadki brzegowe i komunikaty błędów](#przypadki-brzegowe-i-komunikaty-błędów)  
8. [Migracje i seed danych](#migracje-i-seed-danych)  
 

---

## Opis projektu

TeamTaskManager to desktopowa aplikacja napisana w C# wykorzystująca Windows Forms i Entity Framework Core z bazą PostgreSQL. Jej celem jest umożliwienie zarządzania zespołami, użytkownikami oraz zadaniami projektowymi. System wspiera pełen cykl życia zadania: od utworzenia, przez przypisanie członków zespołu, aż po śledzenie postępów w wykonaniu. Ponadto zapewniono rozbudowane uwierzytelnianie, autoryzację (role Admin / Member) oraz walidację danych na wielu poziomach.

---

## Funkcjonalności

1. **Rejestracja i logowanie**  
   - Możliwość rejestracji nowych użytkowników (rola Member domyślnie).  
   - Hasła są przechowywane w postaci hash-ów (BCrypt).  
   - Login za pomocą adresu e-mail i hasła.  
   - Walidacja poprawności adresu e-mail.  

2. **Zarządzanie użytkownikami**  
   - Przeglądanie listy użytkowników: ID, E-mail, Imię, Nazwisko, Kraj, Rola.  
   - Dodawanie nowych użytkowników (tylko rola Admin).  
   - Edycja danych użytkownika (tylko rola Admin).  
   - Usuwanie użytkowników i kaskadowe usuwanie powiązań (zadania, członkostwo w zespołach, itp.).  

3. **Zarządzanie zespołami**  
   - Przeglądanie listy zespołów: ID, Nazwa, Opis, Twórca, liczba członków, liczba zadań.  
   - Dodawanie nowych zespołów (rola Admin lub dowolny Member – w praktyce każdy zalogowany).  
   - Edycja zespołów (tylko właściciel zespołu lub Admin).  
   - Usuwanie zespołów wraz z kaskadowym usuwaniem członków (tylko właściciel lub Admin).  
   - Przypisywanie/odłączanie użytkowników do/z zespołu.  

4. **Zarządzanie zadaniami**  
   - Przeglądanie wszystkich zadań (ID, Tytuł, Opis, Termin, Zespół, Twórca, Obecny status).  
   - Dodawanie nowych zadań (każdy zalogowany).  
     - Automatyczne dodanie wpisu w `Progress` („Zadanie utworzone”).  
     - Możliwość wybrania zespołu, przypisania użytkowników (multipicker).  
   - Edycja istniejących zadań (tylko właściciel lub Admin).  
   - Usuwanie zadań (tylko właściciel lub Admin).  
   - Śledzenie historii postępów (`Progress`): każdy wpis zawiera status, komentarz i znacznik czasu.  
   - Dodawanie nowych wpisów postępu (tylko właściciel lub Admin) – zmiana statusu zadania.  

5. **Role i uprawnienia**  
   - Rola `Admin`: pełny dostęp (dodawanie/edycja/usuwanie użytkowników, zespołów, zadań).  
   - Rola `Member`: może przeglądać listy, dodawać swoje zadania i zespoły, edytować/usuwać tylko swoje zasoby (zadania, zespoły).  
   - Walidacja po stronie UI i w serwisach – sprawdzanie, czy użytkownik ma uprawnienia do danej operacji.  

6. **Walidacja danych**  
   - Adres e-mail: unikalny, weryfikowany w serwisach i formularzach.  
   - Nazwy zespołów: unikalne.  
   - Tytuł zadania: wymagany, max. długość 200 znaków.  
   - Opis zespołu: max. długość 500 znaków.  
   - Opis zadania: max. długość 1000 znaków.  
   - Status zadania i postępu: ograniczona lista wartości (`Nierozpoczęte`, `Nowe`, `W toku`, `Zakończone`).  

7. **Spojrzenie na dane**  
   - Aplikacja automatycznie stosuje migracje EF Core przy starcie, a następnie inicjalizuje przykładowe dane (seed):  
     - Przynajmniej 3 użytkowników (Admin, dwóch Members).  
     - 2 przykładowe zespoły.  
     - Kilka przykładowych zadań.  
     - Kilka wpisów w tabeli `Progress`.  

---

## Technologie

- **Język**: C# 11 (.NET 8.0)  
- **UI**: Windows Forms (WinForms)  
- **ORM**: Entity Framework Core 8.0  
- **Baza danych**: PostgreSQL (Npgsql EF Core Provider)  
- **Dependency Injection i Hosting**: Microsoft.Extensions.Hosting, Microsoft.Extensions.DependencyInjection  
- **Hashowanie haseł**: BCrypt.Net-Next  
- **Konfiguracja**: Microsoft.Extensions.Configuration.Json (plik `appsettings.json`)  

---

## Architektura i struktura projektu

Aby łatwiej zorientować się w kodzie i zależnościach, poniżej znajduje się omówienie poszczególnych warstw, katalogów oraz najważniejszych plików.

### Struktura katalogów

TeamTaskManager/ <- główny katalog projektu
├── TeamTaskManager.csproj <- plik projektu .NET (WinForms, EF Core itp.)
├── appsettings.json <- konfiguracja (ciągł. połączenia, logowanie)
├── Program.cs <- wejście aplikacji (HostBuilder, DI, migracje, seed, uruchomienie UI)
│
├── Data/ <- kontekst EF Core i inicjalizacja seed
│ ├── AppDbContext.cs <- definicja DbContext, konfiguracja modeli w OnModelCreating
│ ├── SeedData.cs <- metoda Initialize tworząca przykładowe dane
│ └── Migrations/ <- folder generowanych migracji EF Core
│ ├── 20250604192517_InitialCreate.cs
│ ├── AppDbContextModelSnapshot.cs
│
├── Models/ <- klasy encji (modele DB)
│ ├── User.cs <- encja User i enum UserRole
│ ├── Team.cs <- encja Team
│ ├── TeamMember.cs <- encja TeamMember (N:M User–Team)
│ ├── TaskItem.cs <- encja TaskItem
│ ├── TaskAssignment.cs <- encja TaskAssignment (N:M TaskItem–User)
│ └── Progress.cs <- encja Progress (historia postępów)
│
├── Services/ <- warstwa usług (serwisy biznesowe)
│ ├── Interfaces/ <- definicje interfejsów usług
│ │ ├── IUserService.cs
│ │ ├── ITeamService.cs
│ │ └── ITaskService.cs
│ │
│ └── Implementations/ <- implementacja serwisów
│ ├── UserService.cs
│ ├── TeamService.cs
│ └── TaskService.cs
│
└── Forms/ <- Windows Forms (UI) i powiązane pliki projektanta
├── ComboBoxItem.cs <- pomocnicza klasa do ComboBox w formularzach
│
├── LoginForm/ <- formularz logowania
│ ├── LoginForm.cs
│ ├── LoginForm.Designer.cs
│ └── LoginForm.resx
│
├── RegisterForm/ <- formularz rejestracji
│ ├── RegisterForm.cs
│ ├── RegisterForm.Designer.cs
│ └── RegisterForm.resx
│
├── MainForm/ <- główny panel aplikacji po zalogowaniu
│ ├── MainForm.cs
│ ├── MainForm.Designer.cs
│ └── MainForm.resx
│
├── UserForm/ <- zarządzanie użytkownikami (CRUD)
│ ├── UserForm.cs
│ ├── UserForm.Designer.cs
│ └── UserForm.resx
│
├── TeamForm/ <- zarządzanie zespołami (CRUD + przypisywanie członków)
│ ├── TeamForm.cs
│ ├── TeamForm.Designer.cs
│ └── TeamForm.resx
│
└── TaskForm/ <- zarządzanie zadaniami (CRUD + historia postępów)
├── TaskForm.cs
├── TaskForm.Designer.cs
└── TaskForm.resx


### Modele danych

1. **User** (`User.cs`)  
   - `Id` (PK, int)  
   - `Email` (string, max. 150, unikalny, wymagany)  
   - `PasswordHash` (string, wymagany)  
   - `FirstName`, `LastName` (string, max. 50, wymagane)  
   - `Country` (string, max. 50, opcjonalne)  
   - `Role` (enum `UserRole`: Admin | Member)  
   - Nawigacyjne kolekcje:  
     - `CreatedTeams` (zespoły utworzone przez użytkownika)  
     - `CreatedTasks` (zadania utworzone przez użytkownika)  
     - `TeamMembers` (relacja N:M z zespołami)  
     - `TaskAssignments` (relacja N:M z zadaniami)  

2. **Team** (`Team.cs`)  
   - `Id` (PK, int)  
   - `Name` (string, max. 100, unikalny, wymagany)  
   - `Description` (string, max. 500, opcjonalne)  
   - `CreatedByUserId` (FK do `User.Id`, wymagane)  
   - Nawigacyjne kolekcje:  
     - `TeamMembers` (relacja N:M do użytkowników)  
     - `TaskItems` (zadania należące do zespołu)  

3. **TeamMember** (`TeamMember.cs`)  
   - `UserId` (PK kompozytowy z `TeamId`, FK do `User.Id`)  
   - `TeamId` (PK kompozytowy z `UserId`, FK do `Team.Id`)  
   - Nawigacje: `User`, `Team`  

4. **TaskItem** (`TaskItem.cs`)  
   - `Id` (PK, int)  
   - `Title` (string, max. 200, wymagany)  
   - `Description` (string, max. 1000, opcjonalne)  
   - `CreatedAt` (DateTime, automatycznie ustawiane w DB na `NOW()`)  
   - `DueDate` (DateTime, wymagane)  
   - `CurrentStatus` (string, max. 50, synchronizowany z ostatnim wpisem w `Progress`)  
   - `CreatedByUserId` (FK do `User.Id`, wymagane)  
   - `TeamId` (FK do `Team.Id`, wymagane)  
   - Nawigacyjne kolekcje:  
     - `TaskAssignments` (relacja N:M do użytkowników)  
     - `Progresses` (historia postępów zadania)  

5. **TaskAssignment** (`TaskAssignment.cs`)  
   - `TaskItemId` (PK kompozytowy z `UserId`, FK do `TaskItem.Id`)  
   - `UserId` (PK kompozytowy z `TaskItemId`, FK do `User.Id`)  
   - Nawigacje: `TaskItem`, `User`  

6. **Progress** (`Progress.cs`)  
   - `Id` (PK, int)  
   - `Status` (string, max. 50, wymagane)  
   - `Comment` (string, max. 500, opcjonalne)  
   - `Timestamp` (DateTime, automatycznie ustawiane w DB na `NOW()`)  
   - `TaskItemId` (FK do `TaskItem.Id`, wymagane)  
   - Nawigacja: `TaskItem`  

### Warstwa dostępu do danych (EF Core)

**AppDbContext** (`AppDbContext.cs`)  
- Dziedziczy po `DbContext`.  
- Właściwości `DbSet<T>` dla wszystkich encji: `Users`, `Teams`, `TeamMembers`, `TaskItems`, `TaskAssignments`, `Progresses`.  
- Konfiguracja w `OnModelCreating(ModelBuilder modelBuilder)`:  
  1. **User**  
     - Indeks unikalny na `Email`.  
     - Wymagane pola (`Email`, `PasswordHash`, `FirstName`, `LastName`).  
     - Maksymalna długość pól.  
     - Relacje 1:N: `User` → `TeamMembers`, `TaskAssignments`, `CreatedTeams`, `CreatedTasks`.  
     - `OnDelete(DeleteBehavior.Cascade)` dla większości, `Restrict` dla `CreatedTeams` i `CreatedTasks`.  
  2. **Team**  
     - Indeks unikalny na `Name`.  
     - Relacja 1:N: `Team` → `TeamMembers`, `TaskItems`.  
     - Relacja 1:N do `CreatedByUser`.  
  3. **TeamMember** (łącze N:M)  
     - Klucz złożony `(UserId, TeamId)`.  
     - Relacje `HasOne`/`WithMany`, kasowanie kaskadowe.  
  4. **TaskItem**  
     - Klucz prosty `Id`.  
     - Wymagane pola: `Title`, `DueDate`.  
     - Domyślna wartość `CreatedAt = NOW()`.  
     - Relacje 1:N: `TaskItem` → `TaskAssignments`, `Progresses`.  
     - Relacja 1: `CreatedByUser`, 1: `Team`.  
  5. **TaskAssignment**  
     - Klucz złożony `(TaskItemId, UserId)`.  
     - Relacje do `TaskItem` i `User`, kaskadowe usuwanie.  
  6. **Progress**  
     - Klucz prosty `Id`.  
     - Wymagane pole `Status`.  
     - Domyślna wartość `Timestamp = NOW()`.  
     - Relacja 1:N do `TaskItem`.  

**SeedData** (`SeedData.cs`)  
- Sprawdza, czy baza jest pusta (czy są jakiekolwiek dane w `Users`, `Teams`, `TaskItems`).  
- Jeżeli pusta, tworzy przykładowych:  
  1. Użytkowników:  
     - Admin (`admin@firma.com`, `Admin123!`), Jan Kowalski, Polska, rola `Admin`.  
     - Member1 (`user1@firma.com`, `User123!`), Anna Nowak, Niemcy, rola `Member`.  
     - Member2 (`user2@firma.com`, `User456!`), Piotr Wiśniewski, Polska, rola `Member`.  
  2. Zespoły (utworzone przez Admin):  
     - `Projekt A` (zespół do rozwoju projektu A).  
     - `Projekt B` (zespół do badań nad projektem B).  
  3. Powiązanie użytkowników z zespołami (`TeamMembers`):  
     - Admin + user1 w `Projekt A`.  
     - user2 w `Projekt B`.  
  4. Kilka przykładowych zadań:  
     - „Założenie repozytorium” (termin za 7 dni, zespół `Projekt A`, utworzone przez Admin, status „W toku”).  
     - „Analiza wymagań” (termin za 14 dni, zespół `Projekt A`, utworzone przez Admin, status „W toku”).  
     - „Prototyp interfejsu” (termin za 10 dni, zespół `Projekt B`, utworzone przez Admin, status „W toku”).  
  5. Przypisanie zadań do użytkowników (`TaskAssignments`).  
  6. Pierwsze wpisy w `Progress`:  
     - Status „W toku” + komentarz „Zainicjowano repozytorium na GitHub” dla zadania „Założenie repozytorium”.  
     - Status „Nowe” + komentarz „Czekamy na spotkanie z interesariuszami” dla „Analiza wymagań”.  

### Serwisy biznesowe

Wszystkie serwisy implementują interfejsy zdefiniowane w `Services/Interfaces`.

1. **IUserService** / **UserService**  
   - `AuthenticateAsync(email, password)` – sprawdza użytkownika po e-mailu, porównuje hasło (BCrypt), zwraca obiekt `User`.  
   - `IsEmailAvailableAsync(email)` – sprawdza unikalność e-maila.  
   - `CreateUserAsync(newUser, rawPassword)` – walidacja e-maila, hashowanie hasła, ustawienie roli, zapis do bazy.  
   - `GetAllAsync()` – pobranie wszystkich użytkowników posortowanych po nazwisku i imieniu.  
   - `GetByIdAsync(userId)` – pobiera użytkownika wraz z powiązaniami (`TeamMembers`, `TaskAssignments`).  
   - `UpdateUserAsync(user)` – aktualizacja: imię, nazwisko, kraj, rola. Nie zmieniana jest tu logika hasła.  
   - `DeleteUserAsync(userId)` – usuwanie użytkownika, jego powiązań (kaskadowo):  
     1. Usunięcie `TeamMembers`, `TaskAssignments` dla tego użytkownika.  
     2. Usunięcie utworzonych przez niego zadań (kaskadowo `TaskAssignments` + `Progresses`).  
     3. Usunięcie utworzonych przez niego zespołów (najpierw usunięcie `TeamMembers` w tych zespołach).  

2. **ITeamService** / **TeamService**  
   - `GetAllAsync()` – pobiera wszystkie zespoły z powiązaniami (`TeamMembers`, `CreatedByUser`, `TaskItems`), sortuje po nazwie.  
   - `GetByIdAsync(teamId)` – pobiera pojedynczy zespół z członkami i zadaniami.  
   - `CreateTeamAsync(team)` – dodaje zespół.  
   - `UpdateTeamAsync(team)` – aktualizuje nazwę i opis zespołu.  
   - `DeleteTeamAsync(teamId)` – usuwa zespół i jego członków (`TeamMembers`).  
   - `UpdateTeamMembersAsync(teamId, userIds)` – synchronizuje członków zespołu: usuwa wszystkie wcześniejsze powiązania, tworzy nowe.  

3. **ITaskService** / **TaskService**  
   - `GetAllAsync()` – pobiera wszystkie zadania z powiązaniami (`Team`, `CreatedByUser`, `TaskAssignments.User`, `Progresses`).  
   - `GetByIdAsync(taskId)` – pobiera zadanie z powiązaniami (jak wyżej), ale filtrowane po Id.  
   - `CreateTaskAsync(task, assignedUserIds)` –  
     1. Ustawia `CreatedAt = DateTime.UtcNow`, `CurrentStatus = "Nierozpoczęte"` (lub podany).  
     2. Konwertuje datę `DueDate` do UTC, zapisuje `TaskItem`.  
     3. Tworzy wpisy `TaskAssignment` dla wybranych userów.  
     4. Dodaje początkowy wpis w `Progress` (status = aktualny status, komentarz „Zadanie utworzone”).  
     5. Zwraca ponownie pobrane zadanie (z wszystkimi powiązaniami).  
   - `UpdateTaskAsync(task, assignedUserIds)` – synchronizuje zmiany:  
     1. Aktualizuje pól zadania (`Title`, `Description`, `DueDate`, `TeamId`, `CurrentStatus`).  
     2. Synchronizuje tabelę `TaskAssignments`: dodaje nowe, usuwa niepotrzebne.  
   - `DeleteTaskAsync(taskId)` – usuwa zadanie oraz powiązane `TaskAssignments` i `Progresses`.  
   - `AddProgressAsync(taskId, progress)` – dodaje wpis w `Progress` (nowy status, komentarz, `Timestamp = UTC now`), aktualizuje `CurrentStatus` w `TaskItem`, zapisuje zmiany.  
   - `GetProgressHistoryAsync(taskId)` – zwraca historię postępów dla danego zadania (posortowane rosnąco po `Timestamp`).  
   - `FilterTasksAsync(teamId?, userId?, dueBefore?, status?)` – (opcjonalnie) filtruje zadania po parametrach.  

### Formularze WinForms (UI)

#### Klasa pomocnicza: ComboBoxItem (`ComboBoxItem.cs`)

- Prosty wrapper do przechowywania par `(Id, Text)` w ComboBoxach.  
- Metoda `ToString()` zwraca `Text`, dzięki czemu to on jest wyświetlany w kontrolce.

#### LoginForm

- Służy do logowania użytkownika.  
- Zawiera pola:  
  - `txtEmail` (TextBox) – wpisanie e-mailu.  
  - `txtPassword` (Masked TextBox / ustawione `UseSystemPasswordChar = true`) – wpisanie hasła.  
  - `btnLogin` (Button) – próba logowania.  
  - `lblError` (Label) – pokaże komunikat błędu (np. puste pola, niepoprawne dane).  
  - `linkRegister` (LinkLabel) – przejście do formularza rejestracji.  
- Logika:  
  1. Walidacja: e-mail i hasło niepuste. Jeśli puste, ustawienie focus i komunikat.  
  2. Wywołanie `IUserService.AuthenticateAsync(email, password)`.  
  3. Jeśli użytkownik poprawny, utworzenie `UserSession { UserId, Email, FullName, Role }`, zwrócenie `DialogResult.OK` i zamknięcie.  
  4. Jeśli niepoprawny, pokazanie komunikatu „Niepoprawny e-mail lub hasło.”.  
  5. Link „Nie mam konta...” otwiera `RegisterForm`. Jeśli rejestracja zwróciła `DialogResult.OK` z utworzoną sesją, LoginForm również zwraca OK i zamyka się (bez dodatkowego logowania).

#### RegisterForm

- Służy do rejestracji nowego użytkownika.  
- Pola:  
  - `txtEmail`, `txtPassword`, `txtFirstName`, `txtLastName`, `txtCountry`.  
  - `btnRegister`, `btnCancel`, `lblError`, `linkHaveAccount` („Mam już konto”).  
- Logika:  
  1. Podstawowa walidacja: sprawdzenie czy pola e-mail, hasło, imię, nazwisko są niepuste. Jeśli puste – komunikat, focus.  
  2. (Opcjonalnie można było dodać dodatkową walidację formatu e-mail).  
  3. Utworzenie obiektu `User { Email, FirstName, LastName, Country, Role = Member }`.  
  4. Wywołanie `IUserService.CreateUserAsync(newUser, rawPassword)`.  
     - Zostanie sprawdzone, czy e-mail jest dostępny, hasło zahashowane, użytkownik zapisany.  
  5. Po pomyślnym utworzeniu, utworzenie `RegisteredUserSession = new UserSession { ... }`, `DialogResult.OK` i zamknięcie.  
  6. Jeśli wyjątek (np. e-mail zajęty), ustawienie `lblError.Text = "Błąd podczas rejestracji: ..."`.  
  7. Link „Mam już konto” ustawia `DialogResult.OK` i zamyka formularz (powrót do LoginForm).

#### MainForm

- Główny panel aplikacji po zalogowaniu.  
- Konstruktor przyjmuje `IUserService`, `ITeamService`, `ITaskService`, `UserSession currentUser`.  
- Elementy UI:  
  - Górny panel (`panelTop`) z `lpWelcome` (Label) – powitanie imieniem, nazwiskiem i rolą użytkownika.  
  - Lewy panel (`panelLeft`) z przyciskami: `btnManageUsers`, `btnManageTeams`, `btnManageTasks`, `btnExit`.  
  - Kolorystyka: biały background, przycisk Exit w kolorze `LightCoral`, inne przyciski w `WhiteSmoke`.  
  - Tooltip informujący, że nie-Admin widzi przycisk „Użytkownicy” ale jest wyłączony wizualnie (szary) i po kliknięciu otworzy UserForm w trybie tylko-do-odczytu.  
- Logika:  
  - W `MainForm_Load` ustawienie napisu powitalnego według `currentUser.FullName` i `currentUser.Role`.  
  - Jeżeli rola != Admin, przycisk `btnManageUsers` zmienia kolor na `LightGray`, tekst i tooltip („Tylko administrator może edytować użytkowników, inni wyświetlają w trybie podglądu”), ale nadal można go kliknąć.  
  - `btnManageUsers_Click` otwiera `UserForm(_userService, _currentUser)`.  
  - `btnManageTeams_Click` otwiera `TeamForm(_teamService, _userService, _currentUser)`.  
  - `btnManageTasks_Click` otwiera `TaskForm(_taskService, _teamService, _userService, _currentUser)`.  
  - `btnExit_Click` zamyka aplikację.  

#### UserForm

- Pozwala na przeglądanie, dodawanie, edytowanie, usuwanie użytkowników.  
- UI:  
  - `dgvUsers` (DataGridView) – lista wszystkich użytkowników. Kolumny: `Id`, `Email`, `FirstName`, `LastName`, `Country`, `Role`. Ukryte kolumny: `PasswordHash`, `TeamMembers`, `TaskAssignments`, `CreatedTeams`, `CreatedTasks`.  
  - `gbDetails` (GroupBox) z `tableLayoutPanel2` zawiera pola: `txtId`, `txtEmail`, `txtFirstName`, `txtLastName`, `txtCountry`, `cmbRole`, `txtPassword`, `lblPassword`.  
  - Przyciski: `btnAdd`, `btnEdit`, `btnSave`, `btnDelete`, `btnCancel`, `lblMode` (wyświetla tryb: „Tryb: Podgląd” / „Dodawanie” / „Edycja”).  
- Logika UI i uprawnienia:  
  1. W `UserForm_Load`:  
     - Wypełnienie `cmbRole` wartościami `Admin`, `Member` (domyślnie `Member`).  
     - Jeżeli `currentUser.Role != Admin`, to:  
       - `btnAdd`, `btnDelete`, `btnEdit` są wyłączone (disabled), zmieniony kolor na `LightGray`.  
     - Wywołanie `LoadUsersAsync()`, ustawia `SetMode(ViewMode.View)`.  
  2. `LoadUsersAsync()`:  
     - Wywołanie `IUserService.GetAllAsync()`.  
     - Ustawienie `dgvUsers.DataSource = users`.  
     - Ukrycie niepotrzebnych kolumn. Stylizacja nagłówków: tło biało-lawendowe, czcionka pogrubiona.  
  3. `dgvUsers_SelectionChanged`:  
     - W trybie `View` lub `Edit` (ale nie w trakcie dodawania), gdy wybrano wiersz, wypełnienie pól szczegółów: `txtId`, `txtEmail`, `txtFirstName`, `txtLastName`, `txtCountry`, `cmbRole.SelectedItem`.  
     - Ukrycie pola hasła.  
     - Ustawienie `lblMode.Text = "Tryb: Podgląd"`.  
  4. `btnAdd_Click`:  
     - Jeżeli nie Admin, komunikat „Brak uprawnień ...”.  
     - Ustawienie `_isAddingNew = true, _isEditing = false`, `SetMode(ViewMode.Add)`, `ClearFields()`.  
  5. `btnEdit_Click`:  
     - Jeżeli nie Admin, komunikat „Brak uprawnień ...”.  
     - Jeśli wiersz wybrany, wypełnienie pól z wybranego `User`.  
     - Ustawienie `_isAddingNew = false, _isEditing = true`, `SetMode(ViewMode.Edit)`, `lblMode = "Tryb: Edycja"`.  
     - Pole hasła pozostaje ukryte (zmiana hasła nie jest obsługiwana w edycji).  
  6. `SetMode(ViewMode mode)`:  
     - **View**:  
       - Wszystkie pola `ReadOnly = true`, `cmbRole.Enabled = false`, `txtPassword.Visible = false`.  
       - `btnAdd.Enabled = (Admin)`, `btnSave.Enabled = false`, `btnDelete.Enabled = (Admin && wiersz wybrany)`, `btnEdit.Enabled = (Admin && wiersz wybrany)`, `btnCancel.Enabled = false`.  
       - Ustawienie kolorów przycisków wg stanu enabled/disabled.  
       - `lblMode.Text = "Tryb: Podgląd"`.  
     - **Add**:  
       - `txtEmail`, `txtFirstName`, `txtLastName`, `txtCountry`, `cmbRole.Enabled = true`, `txtPassword.Visible = true`.  
       - `btnAdd.Enabled = false`, `btnSave.Enabled = true`, `btnDelete.Enabled = false`, `btnEdit.Enabled = false`, `btnCancel.Enabled = true`.  
       - `lblMode.Text = "Tryb: Dodawanie"`.  
     - **Edit**:  
       - Podobnie do Add, ale `txtPassword.Visible = false`.  
       - `btnAdd.Enabled = false`, `btnSave.Enabled = true`, `btnDelete.Enabled = false`, `btnEdit.Enabled = false`, `btnCancel.Enabled = true`.  
       - `lblMode.Text = "Tryb: Edycja"`.  
  7. `ClearFields()`: czyści `txtId`, `txtEmail`, `txtFirstName`, `txtLastName`, `txtCountry`, ustawia `cmbRole.SelectedIndex = 1` (Member), `txtPassword.Text = ""`.  
  8. `btnSave_Click`:  
     - **Add**:  
       - Walidacja: e-mail niepusty, poprawny format (metoda `IsValidEmail`). Jeśli niepoprawny, komunikat i focus.  
       - Pole hasła niepuste, komunikat i focus, jeżeli puste.  
       - Construct `new User { Email = ..., FirstName = ..., LastName = ..., Country = ..., Role = wybór z cmbRole }`.  
       - `string rawPassword = txtPassword.Text`.  
       - `await _userService.CreateUserAsync(newUser, rawPassword)`.  
       - Po sukcesie: `await LoadUsersAsync()`, `SetMode(ViewMode.View)`, `_isAddingNew = false`.  
       - W przypadku wyjątku (np. e-mail zajęty): `MessageBox.Show($"Błąd podczas dodawania użytkownika: {ex.Message}")`.  
     - **Edit**:  
       - Jeżeli nie Admin – komunikat.  
       - Walidacja: e-mail niepusty, poprawny.  
       - `existing = await _userService.GetByIdAsync(userId); existing.Email = ..., existing.FirstName = ...`, `existing.Role = parsowany enum`.  
       - `await _userService.UpdateUserAsync(existing)`, `await LoadUsersAsync()`, `SetMode(ViewMode.View)`, `_isEditing = false`.  
       - W przypadku wyjątku – komunikat.  
  9. `btnDelete_Click`:  
     - Jeżeli nie Admin – komunikat.  
     - Pobranie `userId` z `txtId`.  
     - `DialogResult confirm = MessageBox.Show("Na pewno chcesz usunąć użytkownika? ...", YesNo)`.  
     - Jeżeli Yes: `await _userService.DeleteUserAsync(userId)`, `await LoadUsersAsync()`, `ClearFields()`, `SetMode(ViewMode.View)`.  
     - W przypadku wyjątku – komunikat.  
  10. `btnCancel_Click`:  
      - Reset `_isAddingNew = false, _isEditing = false`, `SetMode(ViewMode.View)`, `ClearFields()`.  

#### TeamForm

- Pozwala na przeglądanie, dodawanie, edytowanie, usuwanie zespołów oraz zarządzanie ich członkami.  
- UI:  
  - `dgvTeams` (DataGridView) – lista zespołów (ID, Nazwa, Opis, Utworzył, Liczba członków, Liczba zadań, CreatedByUserId [ukryte]).  
  - `gbDetails` (GroupBox) z polami: `txtId`, `txtName`, `txtDescription`, `clbMembers` (CheckedListBox wyświetlający wszystkich użytkowników).  
  - Przyciski: `btnAdd`, `btnEdit`, `btnSave`, `btnDelete`, `btnCancel`, `lblMode` (Label trybu).  
- Logika UI i uprawnienia:  
  1. `TeamForm_Load`: `await LoadTeamsAsync()`, `SetMode(ViewMode.View)`.  
  2. `LoadTeamsAsync()`:  
     - Pobranie wszystkich zespołów z serwisu `GetAllAsync()`, wraz z `TeamMembers`, `CreatedByUser`, `TaskItems`.  
     - Utworzenie projekcji anonimowej:  
       ```csharp
       var displayList = teams.Select(t => new {
           t.Id,
           t.Name,
           t.Description,
           CreatedBy = t.CreatedByUser.FirstName + " " + t.CreatedByUser.LastName,
           MemberCount = t.TeamMembers.Count,
           TaskCount = t.TaskItems.Count,
           CreatedByUserId = t.CreatedByUserId
       }).ToList();
       ```  
     - `dgvTeams.DataSource = displayList`.  
     - Stylizacja nagłówków (jak w innych DataGridView).  
     - Zmiana nagłówków kolumn: `CreatedBy` → „Utworzył”, `MemberCount` → „Liczba członków”, `TaskCount` → „Liczba zadań”.  
     - Ukrycie kolumny `CreatedByUserId`.  
  3. `dgvTeams_SelectionChanged`:  
     - Jeśli nie w trakcie dodawania (`_isAddingNew == false`) i nie w trakcie edycji (`_isEditing == false`) i wybrano wiersz, to:  
       - Pobranie `selectedId` z komórki `Id`.  
       - `var team = await _teamService.GetByIdAsync(selectedId)`.  
       - Wypełnienie pól: `txtId.Text = team.Id`, `txtName.Text = team.Name`, `txtDescription.Text = team.Description`.  
       - W trybie „Podgląd”:  
         - Czyszczenie `clbMembers.Items`.  
         - `clbMembers.Enabled = false`.  
         - Jeśli `team.TeamMembers != null`: pobranie wszystkich użytkowników (`await _userService.GetAllAsync()`), dodanie ich do `clbMembers` jako `TeamComboBoxItem(u.Id, u.FirstName + " " + u.LastName)`, ustawienie `Checked` dla tych, którzy są członkami zespołu (`team.TeamMembers.Any(tm => tm.UserId == u.Id)`).  
       - `lblMode.Text = "Tryb: Podgląd"`.  
  4. `btnAdd_Click`:  
     - `_isAddingNew = true`, `_isEditing = false`, `SetMode(ViewMode.Add)`, `ClearFields()`.  
     - `clbMembers.Items.Clear()`, `clbMembers.Enabled = true`.  
     - Pobranie `users = await _userService.GetAllAsync()`, dla każdego `u` dodanie `TeamComboBoxItem(u.Id, ...)` do `clbMembers`.  
  5. `btnEdit_Click`:  
     - Jeśli żaden wiersz nie jest wybrany – `return`.  
     - `selectedId = (int)dgvTeams.SelectedRows[0].Cells["Id"].Value`, `team = await _teamService.GetByIdAsync(selectedId)`.  
     - Sprawdzenie uprawnień:  
       ```csharp
       int createdById = (int)dgvTeams.SelectedRows[0].Cells["CreatedByUserId"].Value;
       if (_currentUser.Role != UserRole.Admin && createdById != _currentUser.UserId)
       {
           MessageBox.Show("Brak uprawnień. ...");
           return;
       }
       ```  
     - `_isAddingNew = false, _isEditing = true`, `SetMode(ViewMode.Edit)`.  
     - Wypełnienie pól: `txtId.Text`, `txtName.Text`, `txtDescription.Text`.  
     - `clbMembers.Items.Clear()`, `clbMembers.Enabled = true`.  
     - Pobranie wszystkich użytkowników (`await _userService.GetAllAsync()`), dodanie ich do `clbMembers` jako `TeamComboBoxItem`, ustawienie `Checked` dla tych, którzy obecnie należą do zespołu.  
     - `lblMode.Text = "Tryb: Edycja"`.  
  6. `SetMode(ViewMode mode)`:  
     - **View**:  
       - `txtId.ReadOnly = true`, `txtName.ReadOnly = true`, `txtDescription.ReadOnly = true`, `clbMembers.Enabled = false`.  
       - `btnAdd.Enabled = true`, `btnSave.Enabled = false`, `btnDelete.Enabled = (wiersz wybrany)`, `btnEdit.Enabled = (wiersz wybrany)`, `btnCancel.Enabled = false`.  
       - `lblMode.Text = "Tryb: Podgląd"`.  
       - Jeżeli wiersz wybrany:  
         ```csharp
         int createdById = (int)dgvTeams.SelectedRows[0].Cells["CreatedByUserId"].Value;
         if (_currentUser.Role != UserRole.Admin && createdById != _currentUser.UserId)
         {
             btnDelete.Enabled = false;
             btnEdit.Enabled = false;
         }
         ```  
     - **Add**:  
       - `txtId.ReadOnly = true`, `txtName.ReadOnly = false`, `txtDescription.ReadOnly = false`, `clbMembers.Enabled = true`.  
       - `btnAdd.Enabled = false`, `btnSave.Enabled = true`, `btnDelete.Enabled = false`, `btnEdit.Enabled = false`, `btnCancel.Enabled = true`.  
       - `lblMode.Text = "Tryb: Dodawanie"`.  
     - **Edit**:  
       - `txtId.ReadOnly = true`, `txtName.ReadOnly = false`, `txtDescription.ReadOnly = false`, `clbMembers.Enabled = true`.  
       - `btnAdd.Enabled = false`, `btnSave.Enabled = true`, `btnDelete.Enabled = false`, `btnEdit.Enabled = false`, `btnCancel.Enabled = true`.  
       - `lblMode.Text = "Tryb: Edycja"`.  
  7. `ClearFields()`:  
     - `txtId.Text = ""`, `txtName.Text = ""`, `txtDescription.Text = ""`, `clbMembers.Items.Clear()`.  
  8. `btnSave_Click`:  
     - **Add**:  
       - Sprawdzenie `string.IsNullOrWhiteSpace(txtName.Text)` → jeśli true:  
         ```csharp
         MessageBox.Show("Pole Nazwa jest wymagane.");
         txtName.Focus();
         return;
         ```  
       - Utworzenie `newTeam = new Team { Name = txtName.Text.Trim(), Description = txtDescription.Text.Trim(), CreatedByUserId = _currentUser.UserId }`.  
       - `var createdTeam = await _teamService.CreateTeamAsync(newTeam)`.  
       - Zebranie wybranych userIds z `clbMembers.CheckedItems`, konwersja do `List<int>`.  
       - Jeśli `selectedUserIds.Count > 0`, `await _teamService.UpdateTeamMembersAsync(createdTeam.Id, selectedUserIds)`.  
       - `await LoadTeamsAsync()`, `SetMode(ViewMode.View)`, `_isAddingNew = false`.  
       - W przypadku wyjątku: `MessageBox.Show($"Błąd podczas dodawania zespołu: {ex.Message}")`.  
     - **Edit**:  
       - Pobranie `teamId` z `txtId`.  
       - Sprawdzenie uprawnień (podobnie jak w `btnEdit_Click`).  
       - `existing = await _teamService.GetByIdAsync(teamId)`.  
       - `existing.Name = txtName.Text.Trim()`, `existing.Description = txtDescription.Text.Trim()`.  
       - `await _teamService.UpdateTeamAsync(existing)`.  
       - Zebranie `selectedUserIds` z `clbMembers.CheckedIndices` (każdy `TeamComboBoxItem`).  
       - `await _teamService.UpdateTeamMembersAsync(teamId, selectedUserIds)`.  
       - `await LoadTeamsAsync()`, `SetMode(ViewMode.View)`, `_isEditing = false`.  
       - W przypadku wyjątku: `MessageBox.Show($"Błąd podczas aktualizacji zespołu: {ex.Message}")`.  
  9. `btnDelete_Click`:  
     - Pobranie `teamId` z `txtId`.  
     - `toDelete = await _teamService.GetByIdAsync(teamId)`.  
     - Sprawdzenie uprawnień:  
       ```csharp
       int createdById = (int)dgvTeams.SelectedRows[0].Cells["CreatedByUserId"].Value;
       if (_currentUser.Role != UserRole.Admin && createdById != _currentUser.UserId) {
           MessageBox.Show("Brak uprawnień. ...");
           return;
       }
       ```  
     - `confirm = MessageBox.Show("Na pewno chcesz usunąć zespół? ...", YesNo)`.  
     - Jeśli `DialogResult.Yes`: `await _teamService.DeleteTeamAsync(teamId)`, `await LoadTeamsAsync()`, `ClearFields()`, `SetMode(ViewMode.View)`.  
     - W przypadku wyjątku: `MessageBox.Show($"Błąd podczas usuwania: {ex.Message}")`.  
  10. `btnCancel_Click`:  
      - `_isAddingNew = false`, `_isEditing = false`, `SetMode(ViewMode.View)`, `ClearFields()`.  

#### TaskForm

- Pozwala na przeglądanie, dodawanie, edytowanie, usuwanie zadań oraz dodawanie wpisów w historii postępów.  
- UI:  
  - `dgvTasks` (DataGridView) – lista zadań (ID, Title, Description, DueDate, TeamId, CreatedByUserId, CurrentStatus, TaskAssignments, Progresses). Nie pokazujemy kolumn `Progresses`, `TaskAssignments`, `CreatedByUserId` (ukrywamy).  
  - `gbTaskDetails` (GroupBox „Szczegóły zadania”) z polami:  
    - `lblId`, `txtId` (ID, readonly).  
    - `lblTitle`, `txtTitle`.  
    - `lblDescription`, `txtDescription` (multiline).  
    - `lblDueDate`, `dtpDueDate` (DateTimePicker, Format = Short).  
    - `lblTeam`, `cmbTeam` (ComboBox z listą zespołów – `TaskComboBoxItem(Id, Name)`), styl - `DropDownList`.  
    - `lblAssignedUsers`, `clbAssignedUsers` (CheckedListBox z listą użytkowników – `TaskComboBoxItem(Id, Imię + Nazwisko)`).  
    - `lblStatus`, `cmbStatus` (ComboBox z dozwolonymi statusami: `Nierozpoczęte`, `Nowe`, `W toku`, `Zakończone`).  
  - `gbProgressHistory` (GroupBox „Historia postępów”) z elementami:  
    - `dgvProgress` (DataGridView) – lista wpisów postępu: kolumny `Status`, `Comment`, `Timestamp`.  
    - `lblProgressStatus`, `cmbProgressStatus` (ComboBox jak wyżej – wybór statusu nowego wpisu).  
    - `txtProgressComment` (TextBox, `PlaceholderText = "Komentarz"`).  
    - `btnAddProgress` (Button „Dodaj postęp”).  
  - Przyciski: `btnAdd`, `btnSave`, `btnDelete`, `btnCancel`, `lblMode` (Label trybu).  
- Logika UI:  
  1. Konstruktor:  
     - Inicjalizacja komponentów.  
     - Ustawienie kolorów (biały background, `dgvTasks.BackgroundColor = WhiteSmoke`, `gbTaskDetails.BackColor = WhiteSmoke`, `gbProgressHistory.BackColor = WhiteSmoke`).  
     - Przygotowanie `cmbStatus` i `cmbProgressStatus`: wypełnienie listy ze statusami (`Nierozpoczęte`, `Nowe`, `W toku`, `Zakończone`), domyślnie `SelectedIndex = 0`.  
  2. `TaskForm_Load`:  
     - `await LoadTasksAsync()`, `await LoadTeamsAsync()`, `await LoadUsersAsync()`, `SetMode(ViewMode.View)`.  
  3. `LoadTasksAsync()`:  
     - Pobranie listy zadań: `var tasks = await _taskService.GetAllAsync()`.  
     - `dgvTasks.DataSource = tasks`.  
     - Ukrycie kolumn: `Progresses`, `TaskAssignments`, `CreatedByUserId`, jeśli istnieją.  
     - Stylizacja nagłówków: `dgvTasks.EnableHeadersVisualStyles = false`, `ColumnHeadersDefaultCellStyle.BackColor = (230,230,250)`, `Font = Bold`.  

  4. `LoadTeamsAsync()`:  
     - `var teams = await _teamService.GetAllAsync()`.  
     - `cmbTeam.Items.Clear()`.  
     - Dla każdego `t` z `teams`, `cmbTeam.Items.Add(new TaskComboBoxItem(t.Id, t.Name))`.  

  5. `LoadUsersAsync()`:  
     - `var users = await _userService.GetAllAsync()`.  
     - `clbAssignedUsers.Items.Clear()`.  
     - Dla każdego `u` z `users`, `clbAssignedUsers.Items.Add(new TaskComboBoxItem(u.Id, $"{u.FirstName} {u.LastName}"))`.  

  6. `dgvTasks_SelectionChanged`:  
     - Jeśli `_isAddingNew == false` i `dgvTasks.SelectedRows.Count > 0`:  
       - `taskItem = dgvTasks.SelectedRows[0].DataBoundItem as TaskItem`.  
       - Jeżeli `taskItem != null`:  
         - Wypełnij pola szczegółów:  
           - `txtId.Text = taskItem.Id`.  
           - `txtTitle.Text = taskItem.Title`.  
           - `txtDescription.Text = taskItem.Description`.  
           - `dtpDueDate.Value = taskItem.DueDate` (local conversion).  
           - Ustawienie `cmbTeam.SelectedIndex`: przeszukaj `cmbTeam.Items`, gdy `((TaskComboBoxItem)Items[i]).Id == taskItem.TeamId`, ustaw `SelectedIndex = i`.  
           - Ustawienie `clbAssignedUsers`: przeszukaj `clbAssignedUsers.Items`, jeśli w `taskItem.TaskAssignments` znajduje się `UserId`, `SetItemChecked(i, true)`.  
           - Wyświetl historię postępów w `dgvProgress`:  
             ```csharp
             dgvProgress.DataSource = taskItem.Progresses
                 .OrderBy(p => p.Timestamp)
                 .Select(p => new { p.Status, p.Comment, p.Timestamp })
                 .ToList();
             ```  
           - Ustaw `cmbStatus.SelectedItem = taskItem.CurrentStatus` (jeśli lista statusów zawiera tę wartość), w przeciwnym razie `SelectedIndex = 0`.  
           - `lblMode.Text = "Tryb: Podgląd"`.  
           - Reset stanu sekcji „Historia postępów”: `cmbProgressStatus.SelectedIndex = 0`, `txtProgressComment.Text = ""`.  

  7. `btnAdd_Click`:  
     - `_isAddingNew = true`, `SetMode(ViewMode.Add)`, `ClearFields()`.  
     - Nowe zadanie domyślnie z `CurrentStatus = "Nierozpoczęte"` → `cmbStatus.SelectedIndex = 0`.  

  8. `btnSave_Click`:  
     - Jeżeli `_isAddingNew == true` (Tryb Dodawanie):  
       1. Sprawdzenie, czy `cmbTeam.SelectedItem` to `TaskComboBoxItem selectedTeam`. Jeśli nie, `return`.  
       2. Utworzenie obiektu `newTask = new TaskItem { Title = txtTitle.Text.Trim(), Description = txtDescription.Text.Trim(), DueDate = dtpDueDate.Value, CreatedByUserId = _currentUser.UserId, TeamId = selectedTeam.Id, CurrentStatus = cmbStatus.SelectedItem.ToString() }`.  
       3. Wywołanie `await _taskService.CreateTaskAsync(newTask, clbAssignedUsers.CheckedItems.Cast<TaskComboBoxItem>().Select(c => c.Id).ToList())`.  
       4. Po sukcesie: `await LoadTasksAsync()`, `SetMode(ViewMode.View)`, `_isAddingNew = false`.  
       5. W przypadku wyjątku: `MessageBox.Show($"Błąd podczas dodawania zadania: {ex.Message}")`.  

     - Jeżeli `_isAddingNew == false` (Tryb Edycja):  
       1. `if (!int.TryParse(txtId.Text, out int taskId)) return;`  
       2. `existing = await _taskService.GetByIdAsync(taskId)`. Jeżeli null, `return`.  
       3. Walidacja uprawnień:  
          ```csharp
          if (_currentUser.Role != UserRole.Admin && existing.CreatedByUserId != _currentUser.UserId) {
              MessageBox.Show("Brak uprawnień. ...");
              return;
          }
          ```  
       4. `if (!(cmbTeam.SelectedItem is TaskComboBoxItem selTeam)) return;`  
       5. Przypisz nowe wartości do `existing`:  
          ```csharp
          existing.Title = txtTitle.Text.Trim();
          existing.Description = txtDescription.Text.Trim();
          existing.DueDate = dtpDueDate.Value;
          existing.TeamId = selTeam.Id;
          existing.CurrentStatus = cmbStatus.SelectedItem.ToString();
          ```  
       6. `var selectedUserIds = clbAssignedUsers.CheckedItems.Cast<TaskComboBoxItem>().Select(c => c.Id).ToList();`  
       7. `await _taskService.UpdateTaskAsync(existing, selectedUserIds);`  
       8. `await LoadTasksAsync(); SetMode(ViewMode.View);`  

  9. `btnDelete_Click`:  
     1. `if (!int.TryParse(txtId.Text, out int taskId)) return;`  
     2. `toDelete = await _taskService.GetByIdAsync(taskId)`. Jeżeli null, `return`.  
     3. Walidacja uprawnień:  
        ```csharp
        if (_currentUser.Role != UserRole.Admin && toDelete.CreatedByUserId != _currentUser.UserId) {
            MessageBox.Show("Brak uprawnień. ...");
            return;
        }
        ```  
     4. `confirm = MessageBox.Show("Na pewno chcesz usunąć zadanie?", YesNo)`.  
     5. Jeśli `DialogResult.Yes`:  
        ```csharp
        try {
            await _taskService.DeleteTaskAsync(taskId);
            await LoadTasksAsync();
            ClearFields();
            SetMode(ViewMode.View);
        } catch (Exception ex) {
            MessageBox.Show($"Błąd podczas usuwania: {ex.Message}");
        }
        ```  

  10. `btnCancel_Click`:  
      - `_isAddingNew = false`, `SetMode(ViewMode.View)`, `ClearFields()`.  

  11. `btnAddProgress_Click`:  
      1. `if (!int.TryParse(txtId.Text, out int taskId)) return;`  
      2. `toUpdate = await _taskService.GetByIdAsync(taskId)`. Jeżeli null, `return`.  
      3. Walidacja uprawnień:  
         ```csharp
         if (_currentUser.Role != UserRole.Admin && toUpdate.CreatedByUserId != _currentUser.UserId) {
             MessageBox.Show("Brak uprawnień. ...");
             return;
         }
         ```  
      4. `string comment = txtProgressComment.Text.Trim(); if (string.IsNullOrEmpty(comment)) return;`  
      5. `string selectedStatus = cmbProgressStatus.SelectedItem.ToString();`  
      6. `var newProg = new Progress { Status = selectedStatus, Comment = comment };`  
      7. `await _taskService.AddProgressAsync(taskId, newProg);`  
      8. Po dodaniu:  
         ```csharp
         await LoadTasksAsync();
         var updated = await _taskService.GetByIdAsync(taskId);
         dgvProgress.DataSource = updated.Progresses
             .OrderBy(p => p.Timestamp)
             .Select(p => new { p.Status, p.Comment, p.Timestamp })
             .ToList();
         if (_allStatuses.Contains(selectedStatus))
             cmbStatus.SelectedItem = selectedStatus;
         txtProgressComment.Text = string.Empty;
         ```  
      9. W przypadku wyjątku: `MessageBox.Show($"Błąd podczas dodawania postępu: {ex.Message}")`.  

  12. `ClearFields()`:  
      - `txtId.Text = ""`, `txtTitle.Text = ""`, `txtDescription.Text = ""`, `dtpDueDate.Value = DateTime.Now`, `cmbTeam.SelectedIndex = -1`.  
      - Odznaczenie wszystkich w `clbAssignedUsers`.  
      - `dgvProgress.DataSource = null`.  
      - `txtProgressComment.Text = ""`, `cmbStatus.SelectedIndex = 0`, `cmbProgressStatus.SelectedIndex = 0`.  

  13. `SetMode(ViewMode mode)`:  
      - **View**:  
        - `txtId.ReadOnly = true`, `txtTitle.ReadOnly = true`, `txtDescription.ReadOnly = true`, `dtpDueDate.Enabled = false`, `cmbTeam.Enabled = false`, `clbAssignedUsers.Enabled = false`, `cmbStatus.Enabled = false`, `cmbProgressStatus.Enabled = true`, `txtProgressComment.Enabled = true`, `btnAddProgress.Enabled = true`.  
        - `btnAdd.Enabled = true`, `btnSave.Enabled = false`, `btnDelete.Enabled = false`, `btnCancel.Enabled = false`, `lblMode.Text = "Tryb: Podgląd"`.  
        - Jeśli wiersz wybrany:  
          - Jeżeli `(_currentUser.Role == Admin) || (sel.CreatedByUserId == _currentUser.UserId)` → `btnDelete.Enabled = true`.  
      - **Add/Edit**:  
        - `txtId.ReadOnly = true`, `txtTitle.ReadOnly = false`, `txtDescription.ReadOnly = false`, `dtpDueDate.Enabled = true`, `cmbTeam.Enabled = true`, `clbAssignedUsers.Enabled = true`, `cmbStatus.Enabled = true`, `cmbProgressStatus.Enabled = true`, `txtProgressComment.Enabled = true`, `btnAddProgress.Enabled = true`.  
        - `btnAdd.Enabled = false`, `btnSave.Enabled = true`, `btnDelete.Enabled = !_isAddingNew`, `btnCancel.Enabled = true`.  
        - `lblMode.Text = (_isAddingNew ? "Tryb: Dodawanie" : "Tryb: Edycja")`.  

- **TaskComboBoxItem** – klasa pomocnicza analogiczna do `ComboBoxItem`, przechowuje `Id, Text`, `ToString()` zwraca `Text`.  

---

### Inicjalizacja i uruchomienie aplikacji

**Program.cs**  
1. Tworzenie `HostBuilder` (Generic Host) z:  
   - Parsowanie `appsettings.json`.  
   - Rejestracja `AppDbContext` z połączeniem do PostgreSQL (`UseNpgsql`).  
   - Rejestracja serwisów: `IUserService → UserService`, `ITeamService → TeamService`, `ITaskService → TaskService`.  
   - Rejestracja formularzy w DI: `LoginForm`, `RegisterForm`.  
   - Rejestracja fabryki dla `MainForm`: `Func<UserSession, MainForm>`, aby przekazywać dynamicznie utworzoną `UserSession`.  

2. `var host = builder.Build();`

3. W scope:  
   - Pobranie `AppDbContext`.  
   - `context.Database.Migrate();` → automatyczne zastosowanie migracji (`InitialCreate`).  
   - `SeedData.Initialize(context);` → zasianie przykładowych danych (jeśli pusta baza).  

4. Konfiguracja WinForms: `Application.SetHighDpiMode`, `Application.EnableVisualStyles`, `Application.SetCompatibleTextRenderingDefault(false)`.  

5. Otwieranie formularzy:  
   - Tworzymy scope DI.  
   - `var registerForm = scope.ServiceProvider.GetRequiredService<RegisterForm>();`  
   - `var dialog = registerForm.ShowDialog();`  
     - Jeśli `DialogResult.OK`:  
       - Otwieramy `LoginForm`.  
       - Jeżeli `loginForm.ShowDialog() == DialogResult.OK`:  
         - Pobieramy `createMain = scope.ServiceProvider.GetRequiredService<Func<UserSession, MainForm>>();`  
         - `var mainForm = createMain(loginForm.LoggedUser);`  
         - `Application.Run(mainForm);`  
     - Jeśli `registerForm` zwróciło `Cancel` → `Application.Exit()`.  

---

## Instalacja i konfiguracja

### Wymagania wstępne

- **.NET 8.0 SDK** (zalecane):  
  - Można pobrać ze strony [.NET Downloads](https://dotnet.microsoft.com/download).  
  - Upewnij się, że wersja .NET 8.0 jest zainstalowana i dostępna w `sdk` (sprawdź `dotnet --list-sdks`).  

- **Visual Studio 2022/2023** (lub nowsze) ze wsparciem dla .NET 8.0 i Windows Forms  
  - Alternatywnie: **Visual Studio Code** + odpowiednie rozszerzenie C# + uruchamianie z linii komend.  

- **PostgreSQL** (dowolna wersja 12+)  
  - Serwer bazy danych uruchomiony lokalnie (lub zdalnie), dostępny przez `Host`, `Port`, `Username`, `Password`.  
  - Narzędzia: **pgAdmin** lub **psql** (opcjonalne) do weryfikacji bazy.  

### Konfiguracja bazy danych

1. **Plik `appsettings.json`** (w katalogu głównym projektu) zawiera:  
   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Host=localhost;Port=5432;Database=TeamTaskManagerDB;Username=postgres;Password=admin;"
     },
     "Logging": {
       "LogLevel": {
         "Default": "Information",
         "Microsoft": "Warning",
         "Microsoft.EntityFrameworkCore": "Warning"
       }
     }
   }
Uzupełnij Host, Port, Database, Username, Password według własnego środowiska.

Utworzenie bazy danych

Można utworzyć bazę ręcznie, np. przez psql:

bash
Kopiuj
Edytuj
psql -U postgres
CREATE DATABASE "TeamTaskManagerDB";
Alternatywnie, EF Core (w Program.cs) utworzy bazę automatycznie przy wywołaniu context.Database.Migrate();, jeżeli użytkownik postgres ma uprawnienia do tworzenia bazy.

Uprawnienia użytkownika

Użytkownik postgres (lub inny wskazany) musi mieć:

Uprawnienie do tworzenia/zastosowania migracji.

Pełny dostęp do schematu (CREATE, ALTER, DROP).

Budowanie i uruchamianie projektu
Otwórz rozwiązanie TeamTaskManager.sln (jeśli istnieje) lub plik TeamTaskManager.csproj w Visual Studio.

Upewnij się, że konfiguracja Startup Project to TeamTaskManager (projekt WinForms).

Przy pierwszym uruchomieniu:

Visual Studio przy kompilacji pobierze potrzebne pakiety (NuGet).

W momencie uruchamiania aplikacji (F5 lub Ctrl+F5), w metodzie Main:

Zostaną zastosowane migracje (jeżeli nie ma bazy lub są nowe migracje).

Zostaną zasiane przykładowe dane (jeśli baza pusta).

Otworzy się RegisterForm.

Uruchom aplikację:

Po kompilacji i uruchomieniu w trybie debug lub Release, pojawi się okno Rejestracji.

Jeżeli chcesz tylko przetestować logowanie, kliknij „Mam już konto” i zostaniesz przeniesiony do LoginForm.

Po poprawnym zalogowaniu, otworzy się MainForm.

Instrukcja obsługi aplikacji
Poniżej znajduje się szczegółowa instrukcja obsługi poszczególnych funkcjonalności. Zawiera również informacje o przypadkach brzegowych i komunikatach błędów.

Rejestracja użytkownika
Otwórz aplikację – wyświetli się formularz RegisterForm.

Uzupełnij pola:

E-mail (pole wymagane, musi być w formacie nazwa@domena, maks. 150 znaków).

Hasło (pole wymagane, nie pokazuje znaków).

Imię (pole wymagane).

Nazwisko (pole wymagane).

Kraj (opcjonalne).

Kliknij Zarejestruj:

Jeżeli pole e-mail jest puste:

Komunikat: „E-mail jest wymagany.”

Ustaw focus na polu e-mail.

Jeżeli pole hasło jest puste:

Komunikat: „Hasło jest wymagane.”

Ustaw focus na polu hasło.

Jeżeli pole Imię jest puste:

Komunikat: „Imię jest wymagane.”

Focus → Imię.

Jeżeli pole Nazwisko jest puste:

Komunikat: „Nazwisko jest wymagane.”

Focus → Nazwisko.

Po podstawowej walidacji, aplikacja wywołuje IUserService.CreateUserAsync().

Jeśli e-mail jest już zajęty (wyjątek InvalidOperationException):

Komunikat: „Błąd podczas rejestracji: Ten e-mail jest już zajęty.”

Użytkownik może zmienić e-mail i spróbować ponownie.

Po udanej rejestracji:

Tworzona jest UserSession (z UserId, Email, FullName, Role = Member).

Formularz zwraca DialogResult.OK i zamyka się.

Jeżeli chcesz zrezygnować, kliknij Anuluj – formularz zamyka się z DialogResult.Cancel i aplikacja kończy działanie.

Po rejestracji (lub bezpośrednio, jeśli klikniesz link “Mam już konto”), otwiera się LoginForm.

Logowanie
Formularz LoginForm:

Pola: txtEmail, txtPassword, btnLogin, lblError, linkRegister.

Uzupełnij:

E-mail (pole wymagane).

Hasło (pole wymagane).

Kliknij Zaloguj (naciśnięcie Enter również aktywuje), lub jeżeli wypełnisz pole i naciśniesz Enter.

Walidacja:

Jeśli txtEmail puste:

lblError.Text = "Podaj e-mail."

Ustaw focus na txtEmail.

Jeśli txtPassword puste:

lblError.Text = "Podaj hasło."

Ustaw focus na txtPassword.

Wywoływane IUserService.AuthenticateAsync(email, password).

Jeśli null (użytkownik nie znaleziony lub hasło niepoprawne):

lblError.Text = "Niepoprawny e-mail lub hasło."

Nie zamyka formularza.

Jeśli poprawny:

Tworzymy UserSession z UserId, Email, FullName, Role.

Formularz zwraca DialogResult.OK, zamyka się.

Po poprawnym logowaniu:

Aplikacja tworzy i otwiera MainForm, przekazując UserSession.

Jeżeli nie masz konta, kliknij link Nie mam konta... – otwiera RegisterForm (możliwość rejestracji). Po rejestracji formularz automatycznie wraca do LoginForm i jeśli rejestracja zakończyła się sukcesem, LoginForm również zamyka się (automatyczne zalogowanie).

Panel główny (MainForm)
Po zalogowaniu otwiera się MainForm, w którym widoczne są cztery przyciski w bocznym panelu oraz powitalny nagłówek.

Nagłówek:

lblWelcome.Text = $"Witaj, {FullName} ({Role})".

Kolor tła: panelTop.BackColor = SteelBlue, kolor tekstu: White.

Przyciski po lewej stronie (panelLeft):

Użytkownicy (btnManageUsers)

Tekst: „Użytkownicy”

Jeżeli rola != Admin:

BackColor = LightGray, ForeColor = DarkGray, Cursor = Cursors.Hand.

Tooltip: „Tylko administrator może edytować użytkowników, inni wyświetlają w trybie podglądu”.

Kliknięcie nadal otwiera UserForm, ale tryb edycji/usuń/dodaj będzie zablokowany.

Zespoły (btnManageTeams)

Tekst: „Zespoły”

Każdy zalogowany (Admin lub Member) może otworzyć TeamForm.

Zadania (btnManageTasks)

Tekst: „Zadania”

Każdy zalogowany może otworzyć TaskForm.

Wyjście (btnExit)

Tekst: „Wyjście”

Kolor tła: LightCoral.

Zamyka aplikację (Application.Exit()).

Kliknięcie któregokolwiek z pierwszych trzech przycisków otwiera odpowiedni formularz w trybie modalnym (ShowDialog()).

Zarządzanie użytkownikami (UserForm)
Otwarcie
MainForm → kliknij Użytkownicy → UserForm ładuje wszystkich użytkowników w dgvUsers.

Widok listy użytkowników
dgvUsers pokazuje kolumny:

Id, Email, FirstName, LastName, Country, Role.

Ukryte kolumny: PasswordHash, TeamMembers, TaskAssignments, CreatedTeams, CreatedTasks.

Nagłówki stylowane: tło z odcieniem lawendowego, pogrubiona czcionka.

Widok szczegółów
Po wybraniu wiersza w dgvUsers:

Wypełniają się pola w gbDetails:

txtId (readonly, ID użytkownika).

txtEmail (readonly w trybie View/Edit, możliwy edytuj tylko w Add).

txtFirstName, txtLastName, txtCountry (readonly w View, edytowalne w Edit/Add).

cmbRole (disabled w View, edytowalne w Add/Edit).

txtPassword i lblPassword nie są widoczne (zmiana hasła nie jest dostępna).

lblMode.Text = "Tryb: Podgląd".

Dodawanie nowego użytkownika
Kliknij Dodaj (btnAdd).

Jeżeli rola != Admin → komunikat „Tylko administrator może dodawać użytkowników.”, zwracamy uwagę i wracamy.

Ustawienie _isAddingNew = true, _isEditing = false.

ClearFields() czyści wszystkie pola w gbDetails.

SetMode(ViewMode.Add):

txtEmail.ReadOnly = false, txtFirstName.ReadOnly = false, txtLastName.ReadOnly = false, txtCountry.ReadOnly = false, cmbRole.Enabled = true.

txtPassword.Visible = true, lblPassword.Visible = true.

btnAdd.Enabled = false, btnSave.Enabled = true, btnDelete.Enabled = false, btnEdit.Enabled = false, btnCancel.Enabled = true.

lblMode.Text = "Tryb: Dodawanie".

Uzupełnij pola:

E-mail (musi być unikalny, poprawny format).

Hasło (niepuste).

Imię, Nazwisko (niepuste).

Kraj (opcjonalny).

Rola (wybierz z cmbRole).

Kliknij Zapisz (btnSave). Mechanizm:

Sprawdź, czy txtEmail jest niepuste. Jeśli puste:

MessageBox.Show("Pole E-mail jest wymagane."), txtEmail.Focus(), return.

Sprawdź poprawność formatu e-mail: IsValidEmail(). Jeśli format błędny:

MessageBox.Show("Nieprawidłowy format e-maila."), txtEmail.Focus(), return.

Sprawdź, czy txtPassword nie jest puste. Jeżeli puste:

MessageBox.Show("Pole Hasło jest wymagane."), txtPassword.Focus(), return.

Utwórz newUser = new User { Email = ..., FirstName = ..., LastName = ..., Country = ..., Role = (parsowanie enum z cmbRole) }.

string rawPassword = txtPassword.Text;

await _userService.CreateUserAsync(newUser, rawPassword):

Jeżeli wyjątek (np. e-mail zajęty):

MessageBox.Show($"Błąd podczas dodawania użytkownika: {ex.Message}").

Po sukcesie:

await LoadUsersAsync(), SetMode(ViewMode.View), _isAddingNew = false.

Edycja istniejącego użytkownika
W dgvUsers wybierz wiersz z użytkownikiem.

Kliknij Edytuj (btnEdit).

Jeżeli rola != Admin → MessageBox.Show("Brak uprawnień. Tylko administrator może edytować użytkowników."), return.

Jeśli wiersz nie jest wybrany → return.

user = dgvUsers.SelectedRows[0].DataBoundItem as User;

Jeżeli user == null → return.

_isAddingNew = false, _isEditing = true, SetMode(ViewMode.Edit).

Wypełnij txtId.Text = user.Id, txtEmail.Text = user.Email, txtFirstName.Text = user.FirstName, txtLastName.Text = user.LastName, txtCountry.Text = user.Country, cmbRole.SelectedItem = user.Role.ToString().

txtPassword.Visible = false, lblPassword.Visible = false (zmiana hasła w tym formularzu nie jest obsługiwana).

lblMode.Text = "Tryb: Edycja".

Dokonaj zmian w polach: Email, FirstName, LastName, Country, Role.

Kliknij Zapisz (btnSave). Logika w trybie Edit:

Jeżeli currentUser.Role != Admin → komunikat i return.

Jeżeli txtEmail jest puste lub !IsValidEmail(txtEmail.Text) →

MessageBox.Show("Nieprawidłowy format e-maila."), txtEmail.Focus(), return.

existing = await _userService.GetByIdAsync(userId). Jeśli null → return.

Nadpisz existing.Email = txtEmail.Text.Trim(), existing.FirstName = txtFirstName.Text.Trim(), existing.LastName = txtLastName.Text.Trim(), existing.Country = txtCountry.Text.Trim(), existing.Role = parsedFrom cmbRole.

await _userService.UpdateUserAsync(existing).

await LoadUsersAsync(), SetMode(ViewMode.View), _isEditing = false.

Jeśli wyjątek podczas aktualizacji → MessageBox.Show($"Błąd podczas aktualizacji: {ex.Message}").

Usuwanie użytkownika
Wybierz wiersz w dgvUsers.

Kliknij Usuń (btnDelete).

Jeżeli rola != Admin → komunikat „Brak uprawnień. ...” → return.

if (!int.TryParse(txtId.Text, out int userId)) return;.

confirm = MessageBox.Show("Na pewno chcesz usunąć użytkownika? Wszystkie powiązane dane zostaną usunięte.", YesNo).

Jeżeli DialogResult.Yes:

await _userService.DeleteUserAsync(userId).

await LoadUsersAsync(), ClearFields(), SetMode(ViewMode.View).

Jeżeli wyjątek → MessageBox.Show($"Błąd podczas usuwania: {ex.Message}").

Anulowanie operacji
Kliknięcie Anuluj (btnCancel) powoduje przerwanie trybu dodawania/edycji:

_isAddingNew = false, _isEditing = false, SetMode(ViewMode.View), ClearFields().

Zarządzanie zespołami (TeamForm)
Otwarcie
MainForm → Zespoły → TeamForm ładuje listę zespołów i ustawia tryb View.

Widok listy zespołów
dgvTeams pokazuje kolumny:

Id, Name, Description, CreatedBy (imię + nazwisko twórcy), MemberCount, TaskCount.

Ukryta kolumna: CreatedByUserId (służy do weryfikacji uprawnień).

Nagłówki stylowane (jak w pozostałych DataGridView).

Widok szczegółów zespołu
Po wybraniu wiersza w dgvTeams:

selectedId = (int)dgvTeams.SelectedRows[0].Cells["Id"].Value.

team = await _teamService.GetByIdAsync(selectedId).

Wypełnij pola:

txtId.Text = team.Id.

txtName.Text = team.Name.

txtDescription.Text = team.Description.

W trybie View:

clbMembers.Items.Clear(), clbMembers.Enabled = false.

Jeśli team.TeamMembers != null:

Pobierz wszystkich użytkowników (await _userService.GetAllAsync()),

Dla każdego u, idx = clbMembers.Items.Add(new TeamComboBoxItem(u.Id, $"{u.FirstName} {u.LastName}")),

Jeśli team.TeamMembers.Any(tm => tm.UserId == u.Id) → clbMembers.SetItemChecked(idx, true).

lblMode.Text = "Tryb: Podgląd".

Dodawanie nowego zespołu
Kliknij Dodaj (btnAdd).

_isAddingNew = true, _isEditing = false, SetMode(ViewMode.Add), ClearFields().

clbMembers.Items.Clear(), clbMembers.Enabled = true.

Pobranie wszystkich użytkowników (await _userService.GetAllAsync()),

Dla każdego u, clbMembers.Items.Add(new TeamComboBoxItem(u.Id, $"{u.FirstName} {u.LastName}")).

lblMode.Text = "Tryb: Dodawanie".

Wypełnij pola:

Nazwa (pole wymagane).

Opis (opcjonalne).

W clbMembers zaznacz (checkbox) użytkowników, którzy mają być członkami zespołu.

Kliknij Zapisz (btnSave). Logika:

Jeśli string.IsNullOrWhiteSpace(txtName.Text):

MessageBox.Show("Pole Nazwa jest wymagane."), txtName.Focus(), return.

Utwórz newTeam = new Team { Name = txtName.Text.Trim(), Description = txtDescription.Text.Trim(), CreatedByUserId = _currentUser.UserId }.

createdTeam = await _teamService.CreateTeamAsync(newTeam).

Pobierz selectedUserIds z clbMembers.CheckedItems (each TeamComboBoxItem).

Jeśli selectedUserIds.Count > 0, await _teamService.UpdateTeamMembersAsync(createdTeam.Id, selectedUserIds).

await LoadTeamsAsync(), SetMode(ViewMode.View), _isAddingNew = false.

W razie wyjątku → MessageBox.Show($"Błąd podczas dodawania zespołu: {ex.Message}").

Edycja istniejącego zespołu
Wybierz wiersz w dgvTeams.

Kliknij Edytuj (btnEdit).

Jeśli żaden nie jest wybrany → return.

selectedId = (int)dgvTeams.SelectedRows[0].Cells["Id"].Value;

team = await _teamService.GetByIdAsync(selectedId); if (team == null) return;

Sprawdzenie uprawnień:

csharp
Kopiuj
Edytuj
int createdById = (int)dgvTeams.SelectedRows[0].Cells["CreatedByUserId"].Value;
if (_currentUser.Role != UserRole.Admin && createdById != _currentUser.UserId) {
    MessageBox.Show("Brak uprawnień. ...");
    return;
}
_isAddingNew = false; _isEditing = true; SetMode(ViewMode.Edit);

txtId.Text = team.Id, txtName.Text = team.Name, txtDescription.Text = team.Description.

clbMembers.Items.Clear(), clbMembers.Enabled = true.

Pobranie wszystkich użytkowników (await _userService.GetAllAsync()),

Dla każdego u, idx = clbMembers.Items.Add(new TeamComboBoxItem(u.Id, $"{u.FirstName} {u.LastName}")),

Jeśli team.TeamMembers.Any(tm => tm.UserId == u.Id) → clbMembers.SetItemChecked(idx, true).

lblMode.Text = "Tryb: Edycja".

Dokonaj zmian w polach:

Nazwa, Opis.

W clbMembers zaznacz / odznacz użytkowników.

Kliknij Zapisz (btnSave). Logika:

Jeśli _isAddingNew (pomijamy, bo to Add).

W przypadku isEditing:

if (!int.TryParse(txtId.Text, out int teamId)) return;

Sprawdzenie uprawnień (tylko Admin albo właściciel – analogicznie jak przy wejściu w Edit).

existing = await _teamService.GetByIdAsync(teamId); if (existing != null) {

existing.Name = txtName.Text.Trim(),

existing.Description = txtDescription.Text.Trim(),

await _teamService.UpdateTeamAsync(existing).

selectedUserIds = clbMembers.CheckedIndices → każdy TeamComboBoxItem→.Id`.

await _teamService.UpdateTeamMembersAsync(teamId, selectedUserIds).

await LoadTeamsAsync(), SetMode(ViewMode.View), _isEditing = false.

}

Wyjątek → MessageBox.Show($"Błąd podczas aktualizacji zespołu: {ex.Message}").

Usuwanie zespołu
Wybierz wiersz w dgvTeams.

Kliknij Usuń (btnDelete).

if (!int.TryParse(txtId.Text, out int teamId)) return;

toDelete = await _teamService.GetByIdAsync(teamId); if (toDelete == null) return;

Sprawdzenie uprawnień:

csharp
Kopiuj
Edytuj
int createdById = (int)dgvTeams.SelectedRows[0].Cells["CreatedByUserId"].Value;
if (_currentUser.Role != UserRole.Admin && createdById != _currentUser.UserId) {
    MessageBox.Show("Brak uprawnień. ...");
    return;
}
confirm = MessageBox.Show("Na pewno chcesz usunąć zespół? Wszystkie powiązane rekordy zostaną usunięte.", YesNo).

Jeżeli DialogResult.Yes:

csharp
Kopiuj
Edytuj
try {
    await _teamService.DeleteTeamAsync(teamId);
    await LoadTeamsAsync();
    ClearFields();
    SetMode(ViewMode.View);
} catch (Exception ex) {
    MessageBox.Show($"Błąd podczas usuwania: {ex.Message}");
}
Anulowanie operacji
Anuluj (btnCancel) → _isAddingNew = false, _isEditing = false, SetMode(ViewMode.View), ClearFields().

Zarządzanie zadaniami (TaskForm)
Otwarcie
MainForm → Zadania → TaskForm ładuje listę zadań, zespoły i użytkowników, ustawia tryb View.

Widok listy zadań
dgvTasks pokazuje kolumny:

Id, Title, Description, CreatedAt, DueDate, CurrentStatus, Team.Name, CreatedByUser.FirstName + " " + CreatedByUser.LastName, TaskAssignments (ukryte), Progresses (ukryte), CreatedByUserId (ukryte).

Ukryte kolumny: Progresses, TaskAssignments, CreatedByUserId.

Stylizacja nagłówków: tło lawendowe, pogrubiona czcionka.

Wybór wiersza w dgvTasks wywołuje dgvTasks_SelectionChanged.

Wyświetlanie szczegółów zadania
dgvTasks_SelectionChanged:

Jeżeli _isAddingNew == false i jest wybrany wiersz:

taskItem = SelectedRows[0].DataBoundItem as TaskItem.

Wypełnij:

txtId.Text = taskItem.Id.

txtTitle.Text = taskItem.Title.

txtDescription.Text = taskItem.Description.

dtpDueDate.Value = taskItem.DueDate.

Ustawienie cmbTeam.SelectedIndex wg taskItem.TeamId.

Wypełnij clbAssignedUsers:

Dla i = 0…count: var cbi = (TaskComboBoxItem)clbAssignedUsers.Items[i];
clbAssignedUsers.SetItemChecked(i, taskItem.TaskAssignments.Any(ta => ta.UserId == cbi.Id)).

Wyświetl historię postępów w dgvProgress:

csharp
Kopiuj
Edytuj
dgvProgress.DataSource = taskItem.Progresses
    .OrderBy(p => p.Timestamp)
    .Select(p => new { p.Status, p.Comment, p.Timestamp })
    .ToList();
Ustaw cmbStatus.SelectedItem = taskItem.CurrentStatus (jeśli jest w _allStatuses), w przeciwnym razie SelectedIndex = 0.

lblMode.Text = "Tryb: Podgląd".

Reset sekcji „Historia postępów”: cmbProgressStatus.SelectedIndex = 0, txtProgressComment.Text = "".

Dodawanie nowego zadania
Kliknij Dodaj (btnAdd).

_isAddingNew = true, SetMode(ViewMode.Add), ClearFields().

cmbStatus.SelectedIndex = 0 (domyślnie „Nierozpoczęte”).

lblMode.Text = "Tryb: Dodawanie".

Wypełnij:

Tytuł (pole wymagane).

Opis (opcjonalne).

Termin (dtpDueDate, domyślnie dzisiaj).

Zespół (cmbTeam – wybór z listy zespołów).

Przypisani użytkownicy (clbAssignedUsers – zaznacz checkbox przy użytkownikach).

Status zadania (cmbStatus – domyślnie „Nierozpoczęte”, można zmienić).

Kliknij Zapisz (btnSave):

Jeżeli ! (cmbTeam.SelectedItem is TaskComboBoxItem selectedTeam) → return.

Utwórz newTask = new TaskItem { Title = ..., Description = ..., DueDate = dtpDueDate.Value, CreatedByUserId = _currentUser.UserId, TeamId = selectedTeam.Id, CurrentStatus = cmbStatus.SelectedItem.ToString() }.

assignedUserIds = clbAssignedUsers.CheckedItems.Cast<TaskComboBoxItem>().Select(c => c.Id).ToList().

await _taskService.CreateTaskAsync(newTask, assignedUserIds):

Ustawia CreatedAt = UTC Now, konwertuje DueDate do UTC, zapisuje TaskItem.

Tworzy TaskAssignments dla assignedUserIds.

Dodaje wpis w Progress (Status = CurrentStatus, Comment = "Zadanie utworzone", Timestamp = UTC Now).

await LoadTasksAsync(), SetMode(ViewMode.View), _isAddingNew = false.

W przypadku błędu: MessageBox.Show($"Błąd podczas dodawania zadania: {ex.Message}").

Edycja istniejącego zadania
Wybierz wiersz w dgvTasks.

Kliknij Zapisz (btnSave) (w trybie View przycisk jest wyłączony).

_isAddingNew == false.

if (!int.TryParse(txtId.Text, out int taskId)) return;.

existing = await _taskService.GetByIdAsync(taskId). Jeżeli null → return.

Walidacja uprawnień:

csharp
Kopiuj
Edytuj
if (_currentUser.Role != UserRole.Admin && existing.CreatedByUserId != _currentUser.UserId) {
    MessageBox.Show("Brak uprawnień. ...");
    return;
}
if (!(cmbTeam.SelectedItem is TaskComboBoxItem selTeam)) return;.

Przypisz nowe wartości do existing:

csharp
Kopiuj
Edytuj
existing.Title = txtTitle.Text.Trim();
existing.Description = txtDescription.Text.Trim();
existing.DueDate = dtpDueDate.Value;
existing.TeamId = selTeam.Id;
existing.CurrentStatus = cmbStatus.SelectedItem.ToString();
selectedUserIds = clbAssignedUsers.CheckedItems.Cast<TaskComboBoxItem>().Select(c => c.Id).ToList().

await _taskService.UpdateTaskAsync(existing, selectedUserIds):

Synchronizuje tabelę TaskAssignments (dodaje / usuwa odpowiednio).

Aktualizuje CurrentStatus.

await LoadTasksAsync(), SetMode(ViewMode.View).

W przypadku wyjątku: MessageBox.Show($"Błąd podczas aktualizacji: {ex.Message}").

Usuwanie zadania
Wybierz wiersz w dgvTasks.

Kliknij Usuń (btnDelete).

if (!int.TryParse(txtId.Text, out int taskId)) return;.

toDelete = await _taskService.GetByIdAsync(taskId). Jeżeli null → return.

Walidacja uprawnień:

csharp
Kopiuj
Edytuj
if (_currentUser.Role != UserRole.Admin && toDelete.CreatedByUserId != _currentUser.UserId) {
    MessageBox.Show("Brak uprawnień. ...");
    return;
}
confirm = MessageBox.Show("Na pewno chcesz usunąć zadanie?", YesNo).

Jeśli DialogResult.Yes:

csharp
Kopiuj
Edytuj
try {
    await _taskService.DeleteTaskAsync(taskId);
    await LoadTasksAsync();
    ClearFields();
    SetMode(ViewMode.View);
} catch (Exception ex) {
    MessageBox.Show($"Błąd podczas usuwania: {ex.Message}");
}
Dodawanie postępu zadania
Wybierz wiersz w dgvTasks (jeśli nie w trybie Add).

Uzupełnij:

Status (cmbProgressStatus) – wybierz nowy status.

Komentarz (txtProgressComment) – pole tekstowe (wymagane, jeśli chcesz dodać wpis).

Kliknij Dodaj postęp (btnAddProgress):

if (!int.TryParse(txtId.Text, out int taskId)) return;.

toUpdate = await _taskService.GetByIdAsync(taskId). Jeżeli null → return.

Walidacja uprawnień:

csharp
Kopiuj
Edytuj
if (_currentUser.Role != UserRole.Admin && toUpdate.CreatedByUserId != _currentUser.UserId) {
    MessageBox.Show("Brak uprawnień. ...");
    return;
}
string comment = txtProgressComment.Text.Trim(); if (string.IsNullOrEmpty(comment)) return;.

string selectedStatus = cmbProgressStatus.SelectedItem.ToString();.

var newProg = new Progress { Status = selectedStatus, Comment = comment };.

await _taskService.AddProgressAsync(taskId, newProg):

Dodaje wpis w Progress (Status = selectedStatus, Comment = comment, Timestamp = UTC Now).

Aktualizuje CurrentStatus w TaskItem na selectedStatus.

Po dodaniu:

csharp
Kopiuj
Edytuj
await LoadTasksAsync();
var updated = await _taskService.GetByIdAsync(taskId);
dgvProgress.DataSource = updated.Progresses
    .OrderBy(p => p.Timestamp)
    .Select(p => new { p.Status, p.Comment, p.Timestamp })
    .ToList();
if (_allStatuses.Contains(selectedStatus)) {
    cmbStatus.SelectedItem = selectedStatus;
}
txtProgressComment.Text = string.Empty;
Jeśli wyjątek → MessageBox.Show($"Błąd podczas dodawania postępu: {ex.Message}").

Anulowanie operacji
Kliknięcie Anuluj (btnCancel) → _isAddingNew = false, SetMode(ViewMode.View), ClearFields().

Walidacja i przypadki brzegowe
W projekcie zastosowano walidację danych na kilku poziomach: po stronie UI (formularzy WinForms) oraz w serwisach (np. kontrola unikalności, uprawnienia).

Walidacja po stronie UI
E-mail (Rejestracja / Logowanie / Edycja użytkownika)

Sprawdzenie, czy pole nie jest puste.

Sprawdzenie formatu:

csharp
Kopiuj
Edytuj
bool IsValidEmail(string email) {
    if (string.IsNullOrWhiteSpace(email)) return false;
    int atPos = email.IndexOf('@');
    if (atPos < 1 || atPos == email.Length - 1) return false;
    int dotPos = email.IndexOf('.', atPos);
    if (dotPos < atPos + 2 || dotPos == email.Length - 1) return false;
    return true;
}
Jeśli format niepoprawny: komunikat „Nieprawidłowy format e-maila.”

Hasło (Rejestracja)

Pole wymagane, niepuste.

Brak walidacji złożoności w UI (można by dodać, ale w obecnej implementacji nie ma).

Imię, Nazwisko (Rejestracja / Edycja użytkownika)

Pola wymagane (niepuste).

Nazwa zespołu

Pole wymagane (niepuste).

Długość max. 100 znaków (konfigurowana w modelu).

Opis zespołu

Długość max. 500 znaków (konfigurowana w modelu).

Tytuł zadania

Pole wymagane (niepuste).

Długość max. 200 znaków (konfigurowana w modelu).

Opis zadania

Długość max. 1000 znaków (konfigurowana w modelu).

Termin zadania (DueDate)

Wybranie daty przez DateTimePicker.

Metody serwisów konwertują tę wartość do UTC.

Status zadania i postępu

ComboBox z ustalonymi wartościami: Nierozpoczęte, Nowe, W toku, Zakończone.

UI wymusza wybór tylko spośród nich (DropDownList).

Członkowie zespołu (CheckedListBox w TeamForm)

Użytkownik może zaznaczyć lub odznaczyć dowolne checkboxy.

Brak dodatkowej walidacji w UI – serwis sprawdzi, czy podane identyfikatory użytkowników są faktycznie w bazie (silnik DB zwróci błąd, jeśli nie ma takiego userId).

Przypisani użytkownicy do zadania (CheckedListBox w TaskForm)

Podobnie jak w TeamForm.

Walidacja w serwisach
UserService

IsEmailAvailableAsync(email) sprawdza unikalność e-maila (zwraca true jeśli nie ma rekordu w DB).

AuthenticateAsync(email, password) – porównanie hasła z hash-em (BCrypt). Jeżeli niepoprawne lub użytkownik nie istnieje, zwraca null.

CreateUserAsync(newUser, rawPassword) – sprawdza ponownie dostępność e-maila (uwaga: w międzyczasie inna część aplikacji mogła utworzyć użytkownika z tym samym e-mailem → wyjątek InvalidOperationException).

UpdateUserAsync(user) – sprawdza, czy użytkownik istnieje. W przeciwnym razie KeyNotFoundException.

DeleteUserAsync(userId) – sprawdza istnienie; jeśli nie istnieje, KeyNotFoundException.

TeamService

GetByIdAsync(teamId) – jeżeli nie istnieje, KeyNotFoundException.

CreateTeamAsync(team) – zapis do DB; nazwa zespołu unikalna dzięki unikalnemu indeksowi w DB. Jeżeli duplikat, migracja/DB zwróci błąd (wyjątek podczas SaveChanges).

UpdateTeamAsync(team) – jeżeli nie istnieje, KeyNotFoundException.

DeleteTeamAsync(teamId) – jeżeli nie istnieje, KeyNotFoundException. Upewnia się, że usuwane są też wpisy w TeamMembers.

UpdateTeamMembersAsync(teamId, userIds) – usuwa wszystkie istniejące powiązania (TeamMembers) z danym teamId, a następnie tworzy nowe dla unikalnych userIds.

TaskService

GetByIdAsync(taskId) – jeśli brak zadania, zwraca null. (Metody wywołujące FindFirstOrDefault).

CreateTaskAsync(task, assignedUserIds) –

Ustawia CreatedAt = DateTime.UtcNow, CurrentStatus = "Nierozpoczęte" (jeżeli pusty).

DueDate = DueDate.ToUniversalTime().

Dodaje task do kontekstu i SaveChanges.

Dodaje TaskAssignment dla każdego z assignedUserIds.

Dodaje Progress z komentarzem „Zadanie utworzone”.

SaveChanges po raz drugi.

Zwraca await GetByIdAsync(task.Id).

UpdateTaskAsync(task, assignedUserIds) –

Pobiera existing z DB (Include(t => t.TaskAssignments)).

Jeśli existing == null → InvalidOperationException("Zadanie nie zostało znalezione.").

Nadpisuje pola: Title, Description, DueDate, TeamId, CurrentStatus.

Synchronizuje TaskAssignments:

currentAssigned = existing.TaskAssignments.Select(ta => ta.UserId).

toAdd = assignedUserIds.Except(currentAssigned), toRemove = currentAssigned.Except(assignedUserIds).

Dodaje TaskAssignment dla toAdd.

Usuwa TaskAssignment dla toRemove.

SaveChanges().

DeleteTaskAsync(taskId) –

Pobranie zadań z Include(t => t.TaskAssignments), Include(t => t.Progresses).

Jeśli null → InvalidOperationException("Zadanie nie zostało znalezione.").

Remove(task), SaveChanges().

AddProgressAsync(taskId, progress) –

task = await FindAsync(taskId), jeśli null → InvalidOperationException.

progress.TaskItemId = taskId, progress.Timestamp = DateTime.UtcNow.

Add(progress).

task.CurrentStatus = progress.Status.

SaveChanges().

GetProgressHistoryAsync(taskId) –

return await _context.Progresses.Where(p => p.TaskItemId == taskId).OrderBy(p => p.Timestamp).ToListAsync();.

FilterTasksAsync(...) – filtruje zapytanie IQueryable<TaskItem> po zadanych parametrach: teamId, userId, dueBefore, status.

Przypadki brzegowe i komunikaty błędów
Rejestracja/Logowanie
Rejestracja

Puste pola → komunikaty o wymaganych polach.

Niepoprawny format e-mail → komunikat „Nieprawidłowy format e-maila.”

E-mail już istnieje w DB → wyjątek z serwisu UserService.CreateUserAsync → MessageBox z treścią „Błąd podczas rejestracji: Ten e-mail jest już zajęty.”

Logowanie

Puste pola → komunikaty „Podaj e-mail.” lub „Podaj hasło.”.

Niepoprawne dane (adres nie istnieje lub hasło nie pasuje) → komunikat „Niepoprawny e-mail lub hasło.”.

Użytkownicy
Edycja / Usuwanie

Jeśli rola != Admin → komunikat „Brak uprawnień. Tylko administrator może ...”.

Jeśli wiersz nie jest wybrany przed Edycją/Usunięciem → brak akcji (bez błędu, metoda zwraca return).

Podczas zapisu:

Puste pole e-mail lub niepoprawny format → komunikat MessageBox.

Jeśli użytkownik z danym ID nie istnieje (np. ktoś inny usunął w międzyczasie) – wyjątek KeyNotFoundException: wyświetlony w MessageBox jako „Błąd podczas aktualizacji: Nie znaleziono użytkownika.”.

Zespoły
Dodawanie

Pusta nazwa → komunikat „Pole Nazwa jest wymagane.”.

Duplikacja nazwy zespołu → wyjątek z EF Core (unikalność Name) → MessageBox z treścią „Błąd podczas dodawania zespołu: ...”.

Edycja / Usuwanie

Jeśli rola != Admin i użytkownik nie jest właścicielem → „Brak uprawnień. Tylko właściciel lub administrator może ...”.

Jeśli zespół nie istnieje (np. już został usunięty) → KeyNotFoundException lub SingleOrDefaultAsync zwróci null → obsłużone przez warunek i return.

Edycja pustej nazwy → „Pole Nazwa jest wymagane.”.

Zadania
Dodawanie

Konieczny wybór zespołu: jeśli cmbTeam.SelectedItem nie jest TaskComboBoxItem, btnSave zwraca return (brak komunikatu, ale można by rozważyć przedstawienie komunikatu „Wybierz zespół.”).

Duplikaty tytułów → dopuszczalne (brak ograniczenia unikalności tytułu).

Edycja / Usuwanie / Dodawanie postępu

Jeśli rola != Admin i użytkownik nie jest właścicielem → „Brak uprawnień. ...”.

Jeśli zadanie nie istnieje (przed pobraniem) → metoda zwraca return.

Dodawanie postępu bez komentarza → return (brak akcji).

Zmiana daty na przeszłość → dopuszczalne (brak walidacji terminu w przeszłości).

Migracje i seed danych
Migracje EF Core (Migrations/20250604192517_InitialCreate.cs)

Tworzenie tabel: Users, Teams, TaskItems, TeamMembers, TaskAssignments, Progresses.

Indeksy: IX_Users_Email (unikalny), IX_Teams_Name (unikalny), klucze obce i indeksy na powiązaniach.

Typy kolumn dostosowane do PostgreSQL (character varying, timestamp with time zone, integer, text).

Snapshot modelu (AppDbContextModelSnapshot.cs)

Automatycznie generowany przez EF Core, odzwierciedla stan modelu.

SeedData

W metodzie Program.Main po utworzeniu hosta i scope:

csharp
Kopiuj
Edytuj
using (var scope = host.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    context.Database.Migrate();
    SeedData.Initialize(context);
}
Dzięki temu przy każdym starcie aplikacji:

Migracje zostaną zastosowane (jeśli są nowe).

Jeżeli baza jest pusta (brak użytkowników, zespołów, zadań) – uzupełniana jest przykładowa zawartość (seed).

