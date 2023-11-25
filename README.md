#include <iostream>
#include <fstream>
#include <vector>
#include <ctime>
#include <iomanip>

using namespace std;

// Clase para representar un artículo en la orden
class OrderItem {
public:
    string itemName;
    double price;
    int quantity;

    OrderItem(string name, double p, int qty) : itemName(name), price(p), quantity(qty) {}
};

// Clase para representar una orden
class Order {
public:
    int ticketNumber;
    string sellerName;
    string date;
    vector<OrderItem> items;
    double discount;
    double tipPercentage;
    double tax;
    bool isCancelled;

    Order(int ticket, string seller, string d) : ticketNumber(ticket), sellerName(seller), date(d), isCancelled(false) {
        discount = 0.0;
        tipPercentage = 0.0;
        tax = 0.0;
    }

    double calculateTotal() const {
        double subtotal = 0.0;
        for (const auto& item : items) {
            subtotal += item.price * item.quantity;
        }

        double total = subtotal - discount;
        total += total * tipPercentage;
        total += total * tax;

        return total;
    }
};

// Clase para el punto de venta del restaurante
class RestaurantPointOfSale {
public:
    vector<Order> orders;

    // Agregar una nueva orden
    void addOrder() {
        int ticketNumber;
        cout << "Ingrese el número de ticket: ";
        cin >> ticketNumber;

        // Verificar si el número de ticket ya existe
        for (const auto& order : orders) {
            if (order.ticketNumber == ticketNumber) {
                cout << "¡Error! El número de ticket ya existe.\n";
                return;
            }
        }

        string sellerName;
        cout << "Ingrese el nombre del vendedor: ";
        cin.ignore(); // Limpiar el buffer de entrada
        getline(cin, sellerName);

        time_t now = time(0);
        tm* localTime = localtime(&now);
        string date = to_string(localTime->tm_mday) + "/" +
            to_string(localTime->tm_mon + 1) + "/" +
            to_string(localTime->tm_year + 1900);

        Order order(ticketNumber, sellerName, date);
        addItemsToOrder(order);

        orders.push_back(order);
        cout << "Orden agregada con éxito.\n";
    }

    // Modificar una orden existente
    void modifyOrder() {
        int ticketNumber;
        cout << "Ingrese el número de ticket a modificar: ";
        cin >> ticketNumber;

        for (auto& order : orders) {
            if (order.ticketNumber == ticketNumber && !order.isCancelled) {
                addItemsToOrder(order);
                cout << "Orden modificada con éxito.\n";
                return;
            }
        }

        cout << "¡Error! No se encontró la orden activa con el número de ticket especificado.\n";
    }

    // Cancelar una orden existente
    void cancelOrder() {
        int ticketNumber;
        cout << "Ingrese el número de ticket a cancelar: ";
        cin >> ticketNumber;

        for (auto& order : orders) {
            if (order.ticketNumber == ticketNumber && !order.isCancelled) {
                order.isCancelled = true;
                cout << "Orden cancelada con éxito.\n";
                return;
            }
        }

        cout << "¡Error! No se encontró la orden activa con el número de ticket especificado.\n";
    }

    // Listar todas las órdenes (activas y canceladas)
    void listOrders() const {
        cout << "Lista de Órdenes:\n";
        for (const auto& order : orders) {
            displayOrderDetails(order);
        }
    }

    // Limpiar la pantalla
    void clearScreen() const {
        // Se puede implementar según el sistema operativo
        // En este ejemplo, simplemente se imprime una serie de líneas en blanco
        for (int i = 0; i < 50; ++i) {
            cout << "\n";
        }
    }

    // Guardar información en un archivo de texto
    void saveToFile() const {
        ofstream outputFile("orders.txt");

        if (outputFile.is_open()) {
            for (const auto& order : orders) {
                outputFile << "Ticket: " << order.ticketNumber << "\n";
                outputFile << "Vendedor: " << order.sellerName << "\n";
                outputFile << "Fecha: " << order.date << "\n";

                outputFile << "Artículos:\n";
                for (const auto& item : order.items) {
                    outputFile << "  - " << item.itemName << " x" << item.quantity << ": $" << item.price << "\n";
                }

                outputFile << "Descuento: $" << order.discount << "\n";
                outputFile << "Propina: " << order.tipPercentage * 100 << "%\n";
                outputFile << "Impuesto: " << order.tax * 100 << "%\n";
                outputFile << "Total: $" << order.calculateTotal() << "\n";
                outputFile << "Estado: " << (order.isCancelled ? "Cancelado" : "Activo") << "\n\n";
            }

            cout << "Información guardada en 'orders.txt'.\n";
        }
        else {
            cout << "¡Error al abrir el archivo para escribir!\n";
        }
    }

    // Función para cargar información desde un archivo de texto
    void loadFromFile() {
        ifstream inputFile("orders.txt");

        if (inputFile.is_open()) {
            orders.clear(); // Limpiar las órdenes existentes

            int ticketNumber;
            string sellerName, date;
            string itemName;
            double price;
            int quantity;
            double discount, tipPercentage, tax;
            bool isCancelled;

            while (inputFile >> ticketNumber >> sellerName >> date >> isCancelled) {
                Order order(ticketNumber, sellerName, date);
                order.isCancelled = isCancelled;

                while (inputFile >> itemName >> price >> quantity) {
                    if (itemName == "Descuento:") {
                        inputFile >> discount;
                        break;
                    }

                    order.items.emplace_back(itemName, price, quantity);
                }

                inputFile >> tipPercentage >> tax;

                order.discount = discount;
                order.tipPercentage = tipPercentage;
                order.tax = tax;

                orders.push_back(order);
            }

            cout << "Información cargada desde 'orders.txt'.\n";
        }
        else {
            cout << "No se encontró el archivo 'orders.txt'.\n";
        }
    }

private:
    // Función auxiliar para agregar artículos a una orden
    void addItemsToOrder(Order& order) const {
        char choice;

        do {
            string itemName;
            double itemPrice;
            int itemQuantity;

            cout << "Ingrese el nombre del artículo: ";
            cin.ignore(); // Limpiar el buffer de entrada
            getline(cin, itemName);

            cout << "Ingrese el precio del artículo: $";
            cin >> itemPrice;

            cout << "Ingrese la cantidad del artículo: ";
            cin >> itemQuantity;

            order.items.emplace_back(itemName, itemPrice, itemQuantity);

            cout << "¿Desea agregar otro artículo? (y/n): ";
            cin >> choice;
        } while (choice == 'y' || choice == 'Y');

        cout << "Ingrese el descuento para la orden ($): ";
        cin >> order.discount;

        cout << "Ingrese el porcentaje de propina (10%, 15%, 20%): ";
        double tipPercentage;
        cin >> tipPercentage;
        order.tipPercentage = tipPercentage / 100.0;

        cout << "Ingrese el porcentaje de impuesto (0%, 5%, 10%): ";
        double taxPercentage;
        cin >> taxPercentage;
        order.tax = taxPercentage / 100.0;
    }

    // Función auxiliar para mostrar los detalles de una orden
    void displayOrderDetails(const Order& order) const {
        cout << "---------------------------------------------\n";
        cout << "Número de Ticket: " << order.ticketNumber << "\n";
        cout << "Vendedor: " << order.sellerName << "\n";
        cout << "Fecha: " << order.date << "\n";

        cout << "Artículos:\n";
        for (const auto& item : order.items) {
            cout << "  - " << item.itemName << " x" << item.quantity << ": $" << item.price << "\n";
        }

        cout << "Descuento: $" << order.discount << "\n";
        cout << "Propina: " << order.tipPercentage * 100 << "%\n";
        cout << "Impuesto: " << order.tax * 100 << "%\n";
        cout << "Total: $" << order.calculateTotal() << "\n";
        cout << "Estado: " << (order.isCancelled ? "Cancelado" : "Activo") << "\n";
    }
};

int main() {
    RestaurantPointOfSale pos;
    pos.loadFromFile(); // Cargar información desde el archivo al inicio

    int choice;

    do {
        cout << "\n----- Punto de Venta de Restaurante -----\n";
        cout << "1. Alta de Ordenes\n";
        cout << "2. Modificar Orden\n";
        cout << "3. Cancelar Orden\n";
        cout << "4. Lista de Ordenes\n";
        cout << "5. Limpiar pantalla\n";
        cout << "6. Guardar y Salir\n";
        cout << "7. Salir sin Guardar\n";
        cout << "Seleccione una opción: ";
        cin >> choice;

        switch (choice) {
        case 1:
            pos.addOrder();
            break;
        case 2:
            pos.modifyOrder();
            break;
        case 3:
            pos.cancelOrder();
            break;
        case 4:
            pos.listOrders();
            break;
        case 5:
            pos.clearScreen();
            break;
        case 6:
            pos.saveToFile();
            break;
        case 7:
            cout << "Saliendo sin guardar.\n";
            break;
        default:
            cout << "Opción no válida. Inténtelo de nuevo.\n";
            break;
        }
    } while (choice != 6 && choice != 7);

    return 0;
}
