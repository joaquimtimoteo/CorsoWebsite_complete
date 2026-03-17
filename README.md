Precisas criar um **projeto separado** apenas com:
- 5 widgets sincronizados
- Salvar geometria da janela

---

## 📁 `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(Lab2GUI3_Widgets VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOMOC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

add_executable(Lab2GUI3_Widgets
    main.cpp
    mainwindow.h
    mainwindow.cpp
)

target_link_libraries(Lab2GUI3_Widgets PRIVATE
    Qt${QT_VERSION_MAJOR}::Widgets)
```

---

## 📁 `main.cpp`

```cpp
#include "mainwindow.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    a.setApplicationName("Lab2GUI3_Widgets");
    a.setOrganizationName("QtLab");
    MainWindow w;
    w.show();
    return a.exec();
}
```

---

## 📁 `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QLabel>
#include <QPlainTextEdit>
#include <QSlider>
#include <QScrollBar>
#include <QSpinBox>
#include <QCheckBox>

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);

protected:
    void closeEvent(QCloseEvent *event) override;

private slots:
    void onCursorChanged();
    void onCheckBoxToggled(bool checked);
    void openSettings();

private:
    // Задание 1 — редактор
    QPlainTextEdit *textEditor;
    QLabel         *cursorLabel;

    // Задание 3 — виджеты
    QSlider    *slider;
    QScrollBar *scrollBar;
    QSpinBox   *spinBox;
    QCheckBox  *checkBox;
    QLabel     *statusLbl;

    void buildUI();
    void buildMenu();
};

#endif
```

---

## 📁 `mainwindow.cpp`

```cpp
#include "mainwindow.h"

#include <QMenuBar>
#include <QToolBar>
#include <QStatusBar>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QGroupBox>
#include <QAction>
#include <QDialog>
#include <QDialogButtonBox>
#include <QSettings>
#include <QCloseEvent>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    setWindowTitle("Lab 2-3: Виджеты и настройки");
    setMinimumSize(480, 400);

    buildUI();
    buildMenu();

    // Задание 1 — позиция курсора
    connect(textEditor, &QPlainTextEdit::cursorPositionChanged,
            this, &MainWindow::onCursorChanged);
    onCursorChanged();

    // Задание 3 — синхронизация
    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            scrollBar, &QScrollBar::setValue);
    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            spinBox,   &QSpinBox::setValue);
    connect(spinBox, QOverload<int>::of(&QSpinBox::valueChanged),
            slider,    &QSlider::setValue);
    connect(checkBox, &QCheckBox::toggled,
            this, &MainWindow::onCheckBoxToggled);

    // Задание 2 — восстановить геометрию
    QSettings s("QtLab", "Lab2GUI3_Widgets");
    if (s.value("saveGeometry", false).toBool()) {
        restoreGeometry(s.value("geo").toByteArray());
        restoreState(s.value("state").toByteArray());
    }
}

// ─────────────────────────────────────────
// ПОСТРОЕНИЕ ИНТЕРФЕЙСА
// ─────────────────────────────────────────
void MainWindow::buildUI()
{
    QWidget     *central = new QWidget(this);
    QVBoxLayout *root    = new QVBoxLayout(central);
    root->setContentsMargins(16, 16, 16, 16);
    root->setSpacing(14);
    setCentralWidget(central);

    // ── Заголовок ──
    QLabel *title = new QLabel(
        "<b>Задание 3: 5 виджетов с сигналами и слотами</b>");
    title->setStyleSheet("font-size:14px; color:#1565C0;");
    root->addWidget(title);

    // ── Группа 1: QPlainTextEdit + cursor ──
    QGroupBox   *grp0  = new QGroupBox(
        "1. QPlainTextEdit — позиция курсора");
    QVBoxLayout *layG0 = new QVBoxLayout(grp0);

    textEditor = new QPlainTextEdit;
    textEditor->setPlaceholderText(
        "Введите текст... Позиция курсора — в строке состояния.");
    textEditor->setFixedHeight(80);
    textEditor->setFont(QFont("Monospace", 10));
    layG0->addWidget(textEditor);
    root->addWidget(grp0);

    // ── Группа 2: Slider / ScrollBar / SpinBox ──
    QGroupBox   *grp1  = new QGroupBox(
        "2. QSlider - QScrollBar - QSpinBox");
    QVBoxLayout *layG1 = new QVBoxLayout(grp1);

    slider    = new QSlider(Qt::Horizontal);
    scrollBar = new QScrollBar(Qt::Horizontal);
    spinBox   = new QSpinBox;

    slider->setRange(0, 100);
    scrollBar->setRange(0, 100);
    spinBox->setRange(0, 100);
    scrollBar->setFixedHeight(18);

    QHBoxLayout *rowSpin = new QHBoxLayout;
    rowSpin->addWidget(new QLabel("QSpinBox:"));
    rowSpin->addWidget(spinBox);
    rowSpin->addStretch();

    layG1->addWidget(new QLabel("QSlider:"));
    layG1->addWidget(slider);
    layG1->addWidget(new QLabel("QScrollBar:"));
    layG1->addWidget(scrollBar);
    layG1->addLayout(rowSpin);
    root->addWidget(grp1);

    // ── Группа 3: CheckBox → Label ──
    QGroupBox   *grp2  = new QGroupBox("3. QCheckBox - QLabel");
    QHBoxLayout *layG2 = new QHBoxLayout(grp2);

    checkBox  = new QCheckBox("Активировать");
    statusLbl = new QLabel("Статус: Выкл");
    statusLbl->setStyleSheet("color:red; font-weight:bold;");

    layG2->addWidget(checkBox);
    layG2->addWidget(statusLbl);
    layG2->addStretch();
    root->addWidget(grp2);

    root->addStretch();

    // ── StatusBar ──
    cursorLabel = new QLabel("Стр: 1  Кол: 1");
    cursorLabel->setStyleSheet("padding:0 10px; color:#1565C0;");
    statusBar()->addPermanentWidget(cursorLabel);
    statusBar()->showMessage("Готово", 2000);
}

// ─────────────────────────────────────────
// МЕНЮ
// ─────────────────────────────────────────
void MainWindow::buildMenu()
{
    QAction *aSet  = new QAction("Параметры...", this);
    QAction *aExit = new QAction("Выход",        this);

    QMenu *mFile = menuBar()->addMenu("Файл");
    mFile->addAction(aExit);

    QMenu *mSet = menuBar()->addMenu("Настройки");
    mSet->addAction(aSet);

    QToolBar *tb = addToolBar("Панель");
    tb->setMovable(false);
    tb->addAction(aSet);

    connect(aSet,  &QAction::triggered, this, &MainWindow::openSettings);
    connect(aExit, &QAction::triggered, this, &QMainWindow::close);
}

// ─────────────────────────────────────────
// ЗАДАНИЕ 1 — Позиция курсора
// ─────────────────────────────────────────
void MainWindow::onCursorChanged()
{
    QTextCursor c   = textEditor->textCursor();
    int ln  = c.blockNumber()  + 1;
    int col = c.columnNumber() + 1;
    cursorLabel->setText(
        QString("Стр: %1  Кол: %2").arg(ln).arg(col));
    statusBar()->showMessage(
        QString("Позиция курсора: строка %1, столбец %2")
            .arg(ln).arg(col), 1500);
}

// ─────────────────────────────────────────
// ЗАДАНИЕ 2 — Настройки геометрии
// ─────────────────────────────────────────
void MainWindow::openSettings()
{
    QDialog dlg(this);
    dlg.setWindowTitle("Настройки");
    dlg.setFixedSize(340, 170);

    auto *lay  = new QVBoxLayout(&dlg);
    auto *grp  = new QGroupBox("Параметры окна");
    auto *glay = new QVBoxLayout(grp);
    auto *chk  = new QCheckBox(
        "Сохранять положение и размер окна");
    auto *hint = new QLabel(
        "При следующем запуске окно откроется\n"
        "на той же позиции и с теми же размерами.");
    hint->setStyleSheet("color:#777; font-size:11px;");
    auto *bbox = new QDialogButtonBox(
        QDialogButtonBox::Ok | QDialogButtonBox::Cancel);

    QSettings s("QtLab", "Lab2GUI3_Widgets");
    chk->setChecked(s.value("saveGeometry", false).toBool());

    glay->addWidget(chk);
    glay->addWidget(hint);
    lay->addWidget(grp);
    lay->addWidget(bbox);

    connect(bbox, &QDialogButtonBox::accepted,
            &dlg, &QDialog::accept);
    connect(bbox, &QDialogButtonBox::rejected,
            &dlg, &QDialog::reject);

    if (dlg.exec() == QDialog::Accepted) {
        QSettings s2("QtLab", "Lab2GUI3_Widgets");
        s2.setValue("saveGeometry", chk->isChecked());
        s2.sync();
        statusBar()->showMessage("Настройки сохранены", 2000);
    }
}

void MainWindow::closeEvent(QCloseEvent *e)
{
    QSettings s("QtLab", "Lab2GUI3_Widgets");
    if (s.value("saveGeometry", false).toBool()) {
        s.setValue("geo",   saveGeometry());
        s.setValue("state", saveState());
        s.sync();
    }
    e->accept();
}

// ─────────────────────────────────────────
// ЗАДАНИЕ 3 — CheckBox → Label
// ─────────────────────────────────────────
void MainWindow::onCheckBoxToggled(bool on)
{
    statusLbl->setText(on ? "Статус: Вкл" : "Статус: Выкл");
    statusLbl->setStyleSheet(
        on ? "color:green; font-weight:bold;"
           : "color:red;   font-weight:bold;");
}
```

---

## Resultado

```
┌──────────────────────────────────────┐
│  Файл   Настройки                    │
├──[⚙ Настройки]───────────────────────┤
│                                      │
│ 1. QPlainTextEdit                    │
│ ┌──────────────────────────────────┐ │
│ │ Введите текст...                 │ │
│ └──────────────────────────────────┘ │
│                                      │
│ 2. QSlider - QScrollBar - QSpinBox   │
│ QSlider:    [━━━━●━━━━━━]            │
│ QScrollBar: [━━━━●━━━━━━]            │
│ QSpinBox:   [ 35 ↑↓ ]               │
│                                      │
│ 3. QCheckBox - QLabel                │
│ ☑ Активировать  Статус: Вкл         │
├──────────────────────────────────────┤
│ Стр: 1  Кол: 1                      │
└──────────────────────────────────────┘
```

Cria um novo projeto no Qt Creator, cola os 4 ficheiros e pressiona **`Ctrl + R`**! 🚀
