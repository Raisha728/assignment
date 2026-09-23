# assignment
import java.io.*;
import java.util.*;

// ===== Menu Item Class =====
class MenuItem {
    private int id;
    private String name;
    private double price;

    public MenuItem(int id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }

    public void setName(String name) { this.name = name; }
    public void setPrice(double price) { this.price = price; }

    public void display() {
        System.out.printf("%d - %s : Rs. %.2f%n", id, name, price);
    }

    public String toFileString() {
        return id + "," + name + "," + price;
    }

    public static MenuItem fromFileString(String line) {
        String[] parts = line.split(",");
        return new MenuItem(Integer.parseInt(parts[0]), parts[1], Double.parseDouble(parts[2]));
    }
}

// ===== Order Class =====
class Order {
    private int orderId;
    private int tableNo;
    private List<MenuItem> items = new ArrayList<>();
    private List<Integer> quantities = new ArrayList<>();

    public Order(int orderId, int tableNo) {
        this.orderId = orderId;
        this.tableNo = tableNo;
    }

    public int getOrderId() { return orderId; }
    public int getTableNo() { return tableNo; }

    public void addItem(MenuItem item, int qty) {
        items.add(item);
        quantities.add(qty);
    }

    public double calculateBill() {
        double total = 0;
        for (int i = 0; i < items.size(); i++) {
            total += items.get(i).getPrice() * quantities.get(i);
        }
        return total;
    }

    public void displayOrder() {
        System.out.println("\nOrder ID: " + orderId + " | Table: " + tableNo);
        for (int i = 0; i < items.size(); i++) {
            System.out.printf("%s x %d = Rs. %.2f%n",
                    items.get(i).getName(),
                    quantities.get(i),
                    items.get(i).getPrice() * quantities.get(i));
        }
        System.out.printf("Total Bill = Rs. %.2f%n", calculateBill());
    }

    public String toFileString() {
        StringBuilder sb = new StringBuilder();
        sb.append(orderId).append(",").append(tableNo).append(",");
        for (int i = 0; i < items.size(); i++) {
            sb.append(items.get(i).getId()).append(":").append(quantities.get(i)).append(";");
        }
        sb.append("Total=").append(calculateBill());
        return sb.toString();
    }
}

// ===== Restaurant Class =====
class Restaurant {
    private List<MenuItem> menu = new ArrayList<>();
    private List<Order> orders = new ArrayList<>();
    private int nextOrderId = 1;
    private Scanner sc = new Scanner(System.in);

    // CRUD for Menu
    public void addMenuItem() {
        System.out.print("Enter Item ID: ");
        int id = sc.nextInt();
        sc.nextLine();
        for (MenuItem m : menu) {
            if (m.getId() == id) {
                System.out.println("Duplicate ID! Item already exists.");
                return;
            }
        }
        System.out.print("Enter Item Name: ");
        String name = sc.nextLine();
        System.out.print("Enter Price: ");
        double price = sc.nextDouble();
        menu.add(new MenuItem(id, name, price));
        System.out.println("Item Added!");
        saveMenuToFile();
    }

    public void viewMenu() {
        System.out.println("\n--- Menu ---");
        for (MenuItem m : menu) {
            m.display();
        }
    }

    public void updateMenuItem() {
        System.out.print("Enter Item ID to update: ");
        int id = sc.nextInt();
        sc.nextLine();
        for (MenuItem m : menu) {
            if (m.getId() == id) {
                System.out.print("Enter new name: ");
                String name = sc.nextLine();
                System.out.print("Enter new price: ");
                double price = sc.nextDouble();
                m.setName(name);
                m.setPrice(price);
                System.out.println("Item updated!");
                saveMenuToFile();
                return;
            }
        }
        System.out.println("Item not found!");
    }

    public void deleteMenuItem() {
        System.out.print("Enter Item ID to delete: ");
        int id = sc.nextInt();
        Iterator<MenuItem> it = menu.iterator();
        while (it.hasNext()) {
            MenuItem m = it.next();
            if (m.getId() == id) {
                it.remove();
                System.out.println("Item deleted!");
                saveMenuToFile();
                return;
            }
        }
        System.out.println("Item not found!");
    }

    // Orders
    public void takeOrder() {
        System.out.print("Enter Table No: ");
        int tableNo = sc.nextInt();
        Order order = new Order(nextOrderId++, tableNo);

        char choice;
        do {
            System.out.print("Enter Item ID: ");
            int id = sc.nextInt();
            System.out.print("Enter Quantity: ");
            int qty = sc.nextInt();

            boolean found = false;
            for (MenuItem m : menu) {
                if (m.getId() == id) {
                    order.addItem(m, qty);
                    found = true;
                }
            }
            if (!found) {
                System.out.println("Invalid Item ID!");
            }
            System.out.print("Add more items? (y/n): ");
            choice = sc.next().charAt(0);
        } while (choice == 'y');

        orders.add(order);
        order.displayOrder();
        saveOrdersToFile(order);
    }

    public void searchOrder() {
        System.out.print("Enter Order ID to search: ");
        int oid = sc.nextInt();
        for (Order o : orders) {
            if (o.getOrderId() == oid) {
                o.displayOrder();
                return;
            }
        }
        System.out.println("Order not found!");
    }

    public void salesReport() {
        double totalSales = 0;
        System.out.println("\n--- Sales Report ---");
        for (Order o : orders) {
            totalSales += o.calculateBill();
        }
        System.out.println("Total Orders: " + orders.size());
        System.out.printf("Total Sales: Rs. %.2f%n", totalSales);
    }

    // File Handling
    private void saveMenuToFile() {
        try (PrintWriter pw = new PrintWriter(new FileWriter("menu.txt"))) {
            for (MenuItem m : menu) {
                pw.println(m.toFileString());
            }
        } catch (IOException e) {
            System.out.println("Error saving menu: " + e.getMessage());
        }
    }

    private void loadMenuFromFile() {
        try (BufferedReader br = new BufferedReader(new FileReader("menu.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                menu.add(MenuItem.fromFileString(line));
            }
        } catch (IOException e) {
            System.out.println("No menu file found, starting fresh.");
        }
    }

    private void saveOrdersToFile(Order order) {
        try (PrintWriter pw = new PrintWriter(new FileWriter("orders.txt", true))) {
            pw.println(order.toFileString());
        } catch (IOException e) {
            System.out.println("Error saving order: " + e.getMessage());
        }
    }

    public void loadOrdersFromFile() {
        try (BufferedReader br = new BufferedReader(new FileReader("orders.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line); // simple display of saved orders
            }
        } catch (IOException e) {
            System.out.println("No orders file found, starting fresh.");
        }
    }

    // Initialize
    public void init() {
        loadMenuFromFile();
        loadOrdersFromFile();
    }
}

// ===== Main Program =====
public class RestaurantBillingSystem {
    public static void main(String[] args) {
        Restaurant r = new Restaurant();
        r.init();
        Scanner sc = new Scanner(System.in);
        int choice;

        do {
            System.out.println("\n===== Restaurant Billing System =====");
            System.out.println("1. Add Menu Item");
            System.out.println("2. View Menu");
            System.out.println("3. Update Menu Item");
            System.out.println("4. Delete Menu Item");
            System.out.println("5. Take Order");
            System.out.println("6. Search Order");
            System.out.println("7. Sales Report");
            System.out.println("8. Exit");
            System.out.print("Enter choice: ");
            choice = sc.nextInt();

            switch(choice) {
                case 1: r.addMenuItem(); break;
                case 2: r.viewMenu(); break;
                case 3: r.updateMenuItem(); break;
                case 4: r.deleteMenuItem(); break;
                case 5: r.takeOrder
