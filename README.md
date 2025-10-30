# 30.10.25

```
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.time.LocalDate;
import java.util.*;
import java.util.List;

// ============== ІНТЕРФЕЙСИ ==============
interface Searchable {
    boolean matches(String query);
}

interface Rentable {
    boolean rent(String userName);
    boolean returnItem();
    boolean isAvailable();
}

interface Displayable {
    String[] toTableRow();
}

// ============== БАЗОВІ КЛАСИ ==============
abstract class LibraryItem implements Searchable, Rentable, Displayable {
    protected String id;
    protected String title;
    protected int year;
    protected boolean available;
    protected String rentedBy;
    protected LocalDate rentDate;

    public LibraryItem(String id, String title, int year) {
        this.id = id;
        this.title = title;
        this.year = year;
        this.available = true;
    }

    @Override
    public boolean rent(String userName) {
        if (!available) {
            return false;
        }
        this.available = false;
        this.rentedBy = userName;
        this.rentDate = LocalDate.now();
        return true;
    }

    @Override
    public boolean returnItem() {
        if (available) {
            return false;
        }
        this.available = true;
        this.rentedBy = null;
        this.rentDate = null;
        return true;
    }

    @Override
    public boolean isAvailable() {
        return available;
    }

    @Override
    public boolean matches(String query) {
        String lowerQuery = query.toLowerCase();
        return id.toLowerCase().contains(lowerQuery) ||
                title.toLowerCase().contains(lowerQuery);
    }

    public abstract String getType();

    // Getters
    public String getId() { return id; }
    public String getTitle() { return title; }
    public int getYear() { return year; }
    public String getRentedBy() { return rentedBy; }
}

class Book extends LibraryItem {
    private String author;
    private int pages;

    public Book(String id, String title, int year, String author, int pages) {
        super(id, title, year);
        this.author = author;
        this.pages = pages;
    }

    @Override
    public String getType() {
        return "Книга";
    }

    @Override
    public String[] toTableRow() {
        return new String[]{
                id,
                getType(),
                title,
                String.valueOf(year),
                "Автор: " + author + ", Сторінок: " + pages,
                available ? "Доступна" : "Орендована",
                rentedBy != null ? rentedBy : "-"
        };
    }

    @Override
    public boolean matches(String query) {
        return super.matches(query) || author.toLowerCase().contains(query.toLowerCase());
    }

    public String getAuthor() {
        return author;
    }

    public int getPages() {
        return pages;
    }
}

class Magazine extends LibraryItem {
    private int issueNumber;
    private String publisher;

    public Magazine(String id, String title, int year, int issueNumber, String publisher) {
        super(id, title, year);
        this.issueNumber = issueNumber;
        this.publisher = publisher;
    }

    @Override
    public String getType() {
        return "Журнал";
    }

    @Override
    public String[] toTableRow() {
        return new String[]{
                id,
                getType(),
                title,
                String.valueOf(year),
                "Випуск: " + issueNumber + ", Видавець: " + publisher,
                available ? "Доступний" : "Орендований",
                rentedBy != null ? rentedBy : "-"
        };
    }

    @Override
    public boolean matches(String query) {
        return super.matches(query) || publisher.toLowerCase().contains(query.toLowerCase());
    }

    public int getIssueNumber() {
        return issueNumber;
    }

    public String getPublisher() {
        return publisher;
    }
}

class DVD extends LibraryItem {
    private String director;
    private int duration;

    public DVD(String id, String title, int year, String director, int duration) {
        super(id, title, year);
        this.director = director;
        this.duration = duration;
    }

    @Override
    public String getType() {
        return "DVD";
    }

    @Override
    public String[] toTableRow() {
        return new String[]{
                id,
                getType(),
                title,
                String.valueOf(year),
                "Режисер: " + director + ", Тривалість: " + duration + " хв",
                available ? "Доступний" : "Орендований",
                rentedBy != null ? rentedBy : "-"
        };
    }

    @Override
    public boolean matches(String query) {
        return super.matches(query) || director.toLowerCase().contains(query.toLowerCase());
    }

    public String getDirector() {
        return director;
    }

    public int getDuration() {
        return duration;
    }
}

// ============== GENERIC-КЛАСИ ==============
class Repository<T extends LibraryItem> {
    private List<T> items;

    public Repository() {
        this.items = new ArrayList<>();
    }

    public void add(T item) {
        items.add(item);
    }

    public boolean remove(String id) {
        for (int i = 0; i < items.size(); i++) {
            if (items.get(i).getId().equals(id)) {
                items.remove(i);
                return true;
            }
        }
        return false;
    }

    public T findById(String id) {
        for (T item : items) {
            if (item.getId().equals(id)) {
                return item;
            }
        }
        return null;
    }

    public List<T> getAll() {
        return new ArrayList<>(items);
    }

    public List<T> search(String query) {
        List<T> results = new ArrayList<>();
        for (T item : items) {
            if (item.matches(query)) {
                results.add(item);
            }
        }
        return results;
    }
}

class LibraryManager {
    private Repository<Book> bookRepository;
    private Repository<Magazine> magazineRepository;
    private Repository<DVD> dvdRepository;

    public LibraryManager() {
        this.bookRepository = new Repository<>();
        this.magazineRepository = new Repository<>();
        this.dvdRepository = new Repository<>();
    }

    public void addItem(LibraryItem item) {
        if (item instanceof Book) {
            bookRepository.add((Book) item);
        } else if (item instanceof Magazine) {
            magazineRepository.add((Magazine) item);
        } else if (item instanceof DVD) {
            dvdRepository.add((DVD) item);
        }
    }

    public List<LibraryItem> getAllItems() {
        List<LibraryItem> allItems = new ArrayList<>();
        allItems.addAll(bookRepository.getAll());
        allItems.addAll(magazineRepository.getAll());
        allItems.addAll(dvdRepository.getAll());
        return allItems;
    }

    public List<LibraryItem> searchAll(String query) {
        List<LibraryItem> allItems = new ArrayList<>();
        allItems.addAll(bookRepository.search(query));
        allItems.addAll(magazineRepository.search(query));
        allItems.addAll(dvdRepository.search(query));
        return allItems;
    }

    public LibraryItem findById(String id) {
        LibraryItem item = bookRepository.findById(id);
        if (item != null) return item;

        item = magazineRepository.findById(id);
        if (item != null) return item;

        item = dvdRepository.findById(id);
        if (item != null) return item;

        return null;
    }
}

// ============== GUI (НЕ ЗМІНЮЙТЕ) ==============
public class LibraryManagementApp extends JFrame {
    private LibraryManager manager;
    private JTable table;
    private DefaultTableModel tableModel;
    private JTextField searchField;
    private JComboBox<String> filterCombo;

    public LibraryManagementApp() {
        manager = new LibraryManager();
        initializeTestData();
        setupUI();
    }

    private void initializeTestData() {
        Book book1 = new Book("B001", "Війна і мир", 1869, "Лев Толстой", 1225);
        manager.addItem(book1);
        Book book2 = new Book("B002", "1984", 1949, "Джордж Орвелл", 328);
        manager.addItem(book2);
        Book book3 = new Book("B003", "Кобзар", 1840, "Тарас Шевченко", 700);
        manager.addItem(book3);

        Magazine mag1 = new Magazine("M001", "National Geographic", 2023, 145, "NG Society");
        manager.addItem(mag1);
        Magazine mag2 = new Magazine("M002", "Vogue", 2023, 10, "Condé Nast");
        manager.addItem(mag2);

        DVD dvd1 = new DVD("D001", "Матриця", 1999, "Вачовскі", 136);
        manager.addItem(dvd1);
        DVD dvd2 = new DVD("D002", "Інтерстеллар", 2014, "Крістофер Нолан", 169);
        manager.addItem(dvd2);
    }

    private void setupUI() {
        setTitle("Система управління бібліотекою");
        setSize(1100, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout(10, 10));

        JPanel topPanel = new JPanel(new FlowLayout(FlowLayout.LEFT, 10, 10));
        topPanel.add(new JLabel("Пошук:"));
        searchField = new JTextField(20);
        topPanel.add(searchField);

        JButton searchBtn = new JButton("Шукати");
        topPanel.add(searchBtn);

        JButton clearSearchBtn = new JButton("Очистити");
        topPanel.add(clearSearchBtn);

        topPanel.add(new JLabel("Фільтр:"));
        filterCombo = new JComboBox<>(new String[]{"Усі", "Книги", "Журнали", "DVD", "Доступні", "Орендовані"});
        topPanel.add(filterCombo);

        add(topPanel, BorderLayout.NORTH);

        String[] columns = {"ID", "Тип", "Назва", "Рік", "Додатково", "Статус", "Орендовано"};
        tableModel = new DefaultTableModel(columns, 0) {
            @Override
            public boolean isCellEditable(int row, int column) {
                return false;
            }
        };
        table = new JTable(tableModel);
        table.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
        table.getColumnModel().getColumn(2).setPreferredWidth(200);
        table.getColumnModel().getColumn(4).setPreferredWidth(250);

        JScrollPane scrollPane = new JScrollPane(table);
        add(scrollPane, BorderLayout.CENTER);

        JPanel bottomPanel = new JPanel(new FlowLayout(FlowLayout.CENTER, 10, 10));
        JButton rentBtn = new JButton("Орендувати");
        JButton returnBtn = new JButton("Повернути");
        JButton addBtn = new JButton("Додати елемент");
        JButton refreshBtn = new JButton("Оновити");
        JButton statsBtn = new JButton("Статистика");

        bottomPanel.add(rentBtn);
        bottomPanel.add(returnBtn);
        bottomPanel.add(addBtn);
        bottomPanel.add(refreshBtn);
        bottomPanel.add(statsBtn);

        add(bottomPanel, BorderLayout.SOUTH);

        searchBtn.addActionListener(e -> {
            String query = searchField.getText().trim();
            if (query.isEmpty()) {
                refreshTable();
            } else {
                List<LibraryItem> results = manager.searchAll(query);
                filterTable(results);
            }
        });

        clearSearchBtn.addActionListener(e -> {
            searchField.setText("");
            filterCombo.setSelectedIndex(0);
            refreshTable();
        });

        rentBtn.addActionListener(e -> rentSelectedItem());

        returnBtn.addActionListener(e -> returnSelectedItem());

        refreshBtn.addActionListener(e -> {
            searchField.setText("");
            filterCombo.setSelectedIndex(0);
            refreshTable();
        });

        addBtn.addActionListener(e -> showAddDialog());

        statsBtn.addActionListener(e -> showStatistics());

        searchField.addActionListener(e -> searchBtn.doClick());

        filterCombo.addActionListener(e -> {
            String selected = (String) filterCombo.getSelectedItem();
            List<LibraryItem> filtered = manager.getAllItems();
            List<LibraryItem> result = new ArrayList<>();

            if (selected.equals("Книги")) {
                for (LibraryItem item : filtered) {
                    if (item instanceof Book) {
                        result.add(item);
                    }
                }
            } else if (selected.equals("Журнали")) {
                for (LibraryItem item : filtered) {
                    if (item instanceof Magazine) {
                        result.add(item);
                    }
                }
            } else if (selected.equals("DVD")) {
                for (LibraryItem item : filtered) {
                    if (item instanceof DVD) {
                        result.add(item);
                    }
                }
            } else if (selected.equals("Доступні")) {
                for (LibraryItem item : filtered) {
                    if (item.isAvailable()) {
                        result.add(item);
                    }
                }
            } else if (selected.equals("Орендовані")) {
                for (LibraryItem item : filtered) {
                    if (!item.isAvailable()) {
                        result.add(item);
                    }
                }
            } else {
                result = filtered;
            }
            filterTable(result);
        });

        refreshTable();
    }

    private void refreshTable() {
        List<LibraryItem> items = manager.getAllItems();
        filterTable(items);
    }

    private void filterTable(List<LibraryItem> items) {
        tableModel.setRowCount(0);
        for (LibraryItem item : items) {
            tableModel.addRow(item.toTableRow());
        }
    }

    private void rentSelectedItem() {
        int selectedRow = table.getSelectedRow();
        if (selectedRow == -1) {
            JOptionPane.showMessageDialog(this, "Будь ласка, виберіть елемент для оренди", "Помилка", JOptionPane.WARNING_MESSAGE);
            return;
        }

        String id = (String) tableModel.getValueAt(selectedRow, 0);
        LibraryItem item = manager.findById(id);

        if (item == null) {
            JOptionPane.showMessageDialog(this, "Помилка: Елемент не знайдено", "Помилка", JOptionPane.ERROR_MESSAGE);
            return;
        }

        if (!item.isAvailable()) {
            JOptionPane.showMessageDialog(this, "Цей елемент вже орендовано користувачем: " + item.getRentedBy(), "Помилка", JOptionPane.WARNING_MESSAGE);
            return;
        }

        String userName = JOptionPane.showInputDialog(this, "Введіть ім'я користувача:", "Оренда елемента", JOptionPane.QUESTION_MESSAGE);

        if (userName != null && !userName.trim().isEmpty()) {
            if (item.rent(userName.trim())) {
                JOptionPane.showMessageDialog(this, "Елемент успішно орендовано!", "Успіх", JOptionPane.INFORMATION_MESSAGE);
                refreshTable();
            }
        }
    }

    private void returnSelectedItem() {
        int selectedRow = table.getSelectedRow();
        if (selectedRow == -1) {
            JOptionPane.showMessageDialog(this, "Будь ласка, виберіть елемент для повернення", "Помилка", JOptionPane.WARNING_MESSAGE);
            return;
        }

        String id = (String) tableModel.getValueAt(selectedRow, 0);
        LibraryItem item = manager.findById(id);

        if (item == null) {
            JOptionPane.showMessageDialog(this, "Помилка: Елемент не знайдено", "Помилка", JOptionPane.ERROR_MESSAGE);
            return;
        }

        if (item.isAvailable()) {
            JOptionPane.showMessageDialog(this, "Цей елемент і так доступний (не орендований)", "Помилка", JOptionPane.WARNING_MESSAGE);
            return;
        }

        int confirm = JOptionPane.showConfirmDialog(this,
                "Повернути елемент, орендований користувачем: " + item.getRentedBy() + "?",
                "Підтвердження повернення",
                JOptionPane.YES_NO_OPTION);

        if (confirm == JOptionPane.YES_OPTION) {
            if (item.returnItem()) {
                JOptionPane.showMessageDialog(this, "Елемент успішно повернено!", "Успіх", JOptionPane.INFORMATION_MESSAGE);
                refreshTable();
            }
        }
    }

    // НЕ ЗМІНЮЙТЕ (складний GUI код)
    private void showAddDialog() {
        String[] types = {"Книга", "Журнал", "DVD"};
        String selectedType = (String) JOptionPane.showInputDialog(this,
                "Виберіть тип елемента:",
                "Додавання елемента",
                JOptionPane.QUESTION_MESSAGE,
                null,
                types,
                types[0]);

        if (selectedType == null) return;

        JPanel panel = new JPanel(new GridLayout(0, 2, 5, 5));
        JTextField idField = new JTextField();
        JTextField titleField = new JTextField();
        JTextField yearField = new JTextField();

        panel.add(new JLabel("ID:"));
        panel.add(idField);
        panel.add(new JLabel("Назва:"));
        panel.add(titleField);
        panel.add(new JLabel("Рік:"));
        panel.add(yearField);

        JTextField field1 = new JTextField();
        JTextField field2 = new JTextField();

        switch (selectedType) {
            case "Книга":
                panel.add(new JLabel("Автор:"));
                panel.add(field1);
                panel.add(new JLabel("Кількість сторінок:"));
                panel.add(field2);
                break;
            case "Журнал":
                panel.add(new JLabel("Номер випуску:"));
                panel.add(field1);
                panel.add(new JLabel("Видавець:"));
                panel.add(field2);
                break;
            case "DVD":
                panel.add(new JLabel("Режисер:"));
                panel.add(field1);
                panel.add(new JLabel("Тривалість (хв):"));
                panel.add(field2);
                break;
        }

        int result = JOptionPane.showConfirmDialog(this, panel,
                "Введіть дані елемента",
                JOptionPane.OK_CANCEL_OPTION,
                JOptionPane.PLAIN_MESSAGE);

        if (result == JOptionPane.OK_OPTION) {
            try {
                String id = idField.getText().trim();
                String title = titleField.getText().trim();
                int year = Integer.parseInt(yearField.getText().trim());

                if (id.isEmpty() || title.isEmpty()) {
                    throw new IllegalArgumentException("ID та назва обов'язкові");
                }

                LibraryItem newItem = null;

                switch (selectedType) {
                    case "Книга":
                        String author = field1.getText().trim();
                        int pages = Integer.parseInt(field2.getText().trim());
                        newItem = new Book(id, title, year, author, pages);
                        break;
                    case "Журнал":
                        int issueNumber = Integer.parseInt(field1.getText().trim());
                        String publisher = field2.getText().trim();
                        newItem = new Magazine(id, title, year, issueNumber, publisher);
                        break;
                    case "DVD":
                        String director = field1.getText().trim();
                        int duration = Integer.parseInt(field2.getText().trim());
                        newItem = new DVD(id, title, year, director, duration);
                        break;
                }

                manager.addItem(newItem);
                refreshTable();
                JOptionPane.showMessageDialog(this,
                        "Елемент успішно додано!",
                        "Успіх",
                        JOptionPane.INFORMATION_MESSAGE);

            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this,
                        "Невірний формат числових даних",
                        "Помилка",
                        JOptionPane.ERROR_MESSAGE);
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(this,
                        "Помилка: " + ex.getMessage(),
                        "Помилка",
                        JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    // НЕ ЗМІНЮЙТЕ (складний код зі статистикою)
    private void showStatistics() {
        List<LibraryItem> allItems = manager.getAllItems();

        int totalCount = allItems.size();

        int bookCount = 0, magazineCount = 0, dvdCount = 0;
        for (LibraryItem item : allItems) {
            if (item instanceof Book) bookCount++;
            else if (item instanceof Magazine) magazineCount++;
            else if (item instanceof DVD) dvdCount++;
        }

        int rentedCount = 0;
        for (LibraryItem item : allItems) {
            if (!item.isAvailable()) rentedCount++;
        }

        double rentedPercentage = totalCount > 0 ? (rentedCount * 100.0 / totalCount) : 0;

        Map<Integer, Integer> yearFrequency = new HashMap<>();
        for (LibraryItem item : allItems) {
            int year = item.getYear();
            yearFrequency.put(year, yearFrequency.getOrDefault(year, 0) + 1);
        }

        int mostPopularYear = 0;
        int maxCount = 0;
        for (Map.Entry<Integer, Integer> entry : yearFrequency.entrySet()) {
            if (entry.getValue() > maxCount) {
                maxCount = entry.getValue();
                mostPopularYear = entry.getKey();
            }
        }

        StringBuilder stats = new StringBuilder();
        stats.append("=== СТАТИСТИКА БІБЛІОТЕКИ ===\n\n");
        stats.append("Всього елементів: ").append(totalCount).append("\n\n");
        stats.append("За типами:\n");
        stats.append("  - Книга: ").append(bookCount).append("\n");
        stats.append("  - Журнал: ").append(magazineCount).append("\n");
        stats.append("  - DVD: ").append(dvdCount).append("\n\n");
        stats.append("Орендовано: ").append(rentedCount)
                .append(" (").append(String.format("%.1f", rentedPercentage)).append("%)\n");
        stats.append("Доступно: ").append(totalCount - rentedCount).append("\n\n");

        if (maxCount > 0) {
            stats.append("Найпопулярніший рік видання: ")
                    .append(mostPopularYear)
                    .append(" (").append(maxCount).append(" елементів)");
        }

        JTextArea textArea = new JTextArea(stats.toString());
        textArea.setEditable(false);
        textArea.setFont(new Font(Font.MONOSPACED, Font.PLAIN, 12));

        JScrollPane scrollPane = new JScrollPane(textArea);
        scrollPane.setPreferredSize(new Dimension(400, 300));

        JOptionPane.showMessageDialog(this,
                scrollPane,
                "Статистика бібліотеки",
                JOptionPane.INFORMATION_MESSAGE);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
            } catch (Exception e) {
                e.printStackTrace();
            }

            LibraryManagementApp app = new LibraryManagementApp();
            app.setLocationRelativeTo(null);
            app.setVisible(true);
        });
    }


```
}
