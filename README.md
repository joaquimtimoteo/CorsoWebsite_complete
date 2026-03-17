## Construtor completo corrigido

Substitui o construtor inteiro no `mainwindow.cpp`:

```cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    setWindowTitle("Lab 2-3: Приложение с GUI");
    setMinimumSize(820, 520);

    buildUI();
    buildMenu();

    // Задание 1 — позиция курсора
    connect(textEditor, &QPlainTextEdit::cursorPositionChanged,
            this, &MainWindow::onCursorChanged);
    onCursorChanged();

    // Задание 3 — синхронизация виджетов
    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            scrollBar, &QScrollBar::setValue);
    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            spinBox, &QSpinBox::setValue);
    connect(spinBox, QOverload<int>::of(&QSpinBox::valueChanged),
            slider, &QSlider::setValue);
    connect(checkBox, &QCheckBox::toggled,
            this, &MainWindow::onCheckBoxToggled);

    // Задание 2 — восстановить геометрию окна
    QSettings s("QtLab", "Lab2GUI3");
    if (s.value("saveGeometry", false).toBool()) {
        restoreGeometry(s.value("geo").toByteArray());
        restoreState(s.value("state").toByteArray());
    }

    // Демо-данные sin(x)
    for (int i = 0; i <= 62; i++) {
        double x = i * 0.1;
        m_xData << x;
        m_yData << std::sin(x);
    }
    drawGraph();
}
```

---

## Resumo das 3 funções que precisas ter iguais

Para garantir que o `QSettings` usa sempre o mesmo nome em todos os lugares, as 3 funções devem estar assim:

```cpp
// ── No construtor ──
QSettings s("QtLab", "Lab2GUI3");
if (s.value("saveGeometry", false).toBool()) {
    restoreGeometry(s.value("geo").toByteArray());
    restoreState(s.value("state").toByteArray());
}

// ── No openSettings() ──
QSettings s("QtLab", "Lab2GUI3");
// ... ler e gravar com s.sync()

// ── No closeEvent() ──
QSettings s("QtLab", "Lab2GUI3");
// ... gravar com s.sync()
```

O problema anterior era que cada função usava um `QSettings` diferente — agora todas usam `"QtLab", "Lab2GUI3"` e ficam sincronizadas. 🚀

## Construtor completo corrigido

Substitui o construtor inteiro no `mainwindow.cpp`:

```cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    setWindowTitle("Lab 2-3: Приложение с GUI");
    setMinimumSize(820, 520);

    buildUI();
    buildMenu();

    // Задание 1 — позиция курсора
    connect(textEditor, &QPlainTextEdit::cursorPositionChanged,
            this, &MainWindow::onCursorChanged);
    onCursorChanged();

    // Задание 3 — синхронизация виджетов
    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            scrollBar, &QScrollBar::setValue);
    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            spinBox, &QSpinBox::setValue);
    connect(spinBox, QOverload<int>::of(&QSpinBox::valueChanged),
            slider, &QSlider::setValue);
    connect(checkBox, &QCheckBox::toggled,
            this, &MainWindow::onCheckBoxToggled);

    // Задание 2 — восстановить геометрию окна
    QSettings s("QtLab", "Lab2GUI3");
    if (s.value("saveGeometry", false).toBool()) {
        restoreGeometry(s.value("geo").toByteArray());
        restoreState(s.value("state").toByteArray());
    }

    // Демо-данные sin(x)
    for (int i = 0; i <= 62; i++) {
        double x = i * 0.1;
        m_xData << x;
        m_yData << std::sin(x);
    }
    drawGraph();
}
```

---

## Resumo das 3 funções que precisas ter iguais

Para garantir que o `QSettings` usa sempre o mesmo nome em todos os lugares, as 3 funções devem estar assim:

```cpp
// ── No construtor ──
QSettings s("QtLab", "Lab2GUI3");
if (s.value("saveGeometry", false).toBool()) {
    restoreGeometry(s.value("geo").toByteArray());
    restoreState(s.value("state").toByteArray());
}

// ── No openSettings() ──
QSettings s("QtLab", "Lab2GUI3");
// ... ler e gravar com s.sync()

// ── No closeEvent() ──
QSettings s("QtLab", "Lab2GUI3");
// ... gravar com s.sync()
```

O problema anterior era que cada função usava um `QSettings` diferente — agora todas usam `"QtLab", "Lab2GUI3"` e ficam sincronizadas. 🚀
