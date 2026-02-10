# prgAssignment
 


//==========================================================
// Student Number : 
// Student Name : 
// Partner Name : 
//==========================================================  



using S10272786F_PRG2Assignment;
using System;
using System.Collections;
using System.Collections.Generic;
using System.Diagnostics;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text.RegularExpressions;

List<Restaurant> RestaurantList = new List<Restaurant>();
Stack<Order> RefundStack = new Stack<Order>();

List<Customer> customers = new();
Dictionary<string, Restaurant> restaurantById = new();
Dictionary<string, Customer> customerByEmail = new();
Dictionary<int, string> orderRestaurantMap = new();

int nextOrderId = 1;
//customer path? 
string ordersFilePath = @"C:\NP 2025 SEM 2\PRG2 Programming 2\S10274330_PRG2Assignment\S10274330_PRG2Assignment\orders.csv";
string path = @"C:\NP 2025 SEM 2\PRG2 Programming 2\S10274330_PRG2Assignment\S10274330_PRG2Assignment\customers.csv";

try
{
    LoadRestaurants();
    LoadFoodItems();
    LoadCustomers(path);
    LoadOrders(ordersFilePath);

    Console.WriteLine("Welcome to the Gruberoo Food Delivery System");

    // Main menu loop
    while (true)
    {
        try
        {
            DisplayMainMenu();
            Console.Write("Enter your choice: ");
            string input = Console.ReadLine();

            if (!int.TryParse(input, out int option))
            {
                Console.WriteLine("Invalid input. Please enter a number.");
                continue;
            }

            if (option == 1)
            {
                ListAllRestaurantsAndMenuItems();
            }
            else if (option == 2)
            {
                ListAllOrders();
            }
            else if (option == 3)
            {
                CreateNewOrder();
            }
            else if (option == 4)
            {
                ProcessOrder();
            }
            else if (option == 5)
            {
                ModifyExistingOrder();
            }
            else if (option == 6)
            {
                DeleteOrder();
            }
            else if (option == 0)
            {
                break;
            }
            else
            {
                Console.WriteLine("Invalid option. Please select 0-6.");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"An error occurred: {ex.Message}");
            Console.WriteLine("Please try again.");
        }
    }
}
catch (Exception ex)
{
    Console.WriteLine($"Fatal error: {ex.Message}");
    Console.WriteLine("The application will now exit.");
}

void DisplayMainMenu()
{
    Console.WriteLine("===== Gruberoo Food Delivery System =====" +
        "\n1. List all restaurants and menu items" +
        "\n2. List all orders" +
        "\n3. Create a new order" +
        "\n4. Process an order" +
        "\n5. Modify an existing order" +
        "\n6. Delete an existing order" +
        "\n0. Exit");
}



// basic feature 1 : Gui Ru 

void LoadRestaurants()
{
    try
    {
        string restaurantPath = @"C:\NP 2025 SEM 2\PRG2 Programming 2\S10274330_PRG2Assignment\S10274330_PRG2Assignment\restaurants.csv";
        if (!File.Exists(restaurantPath))
        {
            Console.WriteLine("restaurants.csv file not found!");
            return;
        }

        using (StreamReader sr = new StreamReader(restaurantPath))
        {
            string heading = sr.ReadLine();
            string line;
            int count = 0;
            while ((line = sr.ReadLine()) != null)
            {
                try
                {
                    string[] data = line.Split(',');
                    if (data.Length < 3)
                    {
                        Console.WriteLine($"Skipping invalid restaurant line: {line}");
                        continue;
                    }

                    string restaurantID = data[0].Trim();
                    string name = data[1].Trim();
                    string email = data[2].Trim();

                    Restaurant r = new Restaurant(restaurantID, name, email);
                    RestaurantList.Add(r);
                    restaurantById[restaurantID] = r;
                    count++;
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Error processing restaurant line: {ex.Message}");
                }
            }
            Console.WriteLine($"{count} restaurants loaded!");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading restaurants: {ex.Message}");
    }
}

void LoadFoodItems()
{
    try
    {
        string FoodItemPath = @"C:\NP 2025 SEM 2\PRG2 Programming 2\S10274330_PRG2Assignment\S10274330_PRG2Assignment\fooditems.csv";

        if (!File.Exists(FoodItemPath))
        {
            Console.WriteLine("fooditems.csv file not found!");
            return;
        }

        using (StreamReader sr1 = new StreamReader(FoodItemPath))
        {
            string heading = sr1.ReadLine();
            string line;
            int rows = 0;

            while ((line = sr1.ReadLine()) != null)
            {
                try
                {
                    string[] data = line.Split(',');
                    if (data.Length < 4)
                    {
                        Console.WriteLine($"Skipping invalid food item line: {line}");
                        continue;
                    }

                    string restaurantID = data[0].Trim();
                    string itemName = data[1].Trim();
                    string desc = data[2].Trim();

                    if (!double.TryParse(data[3].Trim(), out double price))
                    {
                        Console.WriteLine($"Invalid price for {itemName}, skipping");
                        continue;
                    }

                    Restaurant restaurant = FindRestaurant(restaurantID);

                    if (restaurant != null)
                    {
                        // Create food item with empty customise field
                        FoodItem foodItem = new FoodItem(itemName, desc, price, "");

                        // Check if restaurant has a menu, if not create one
                        if (restaurant.Menus.Count == 0)
                        {
                            restaurant.AddMenu(new Menu("M001", "Main Menu"));
                        }

                        // Add food item to the first menu
                        restaurant.Menus[0].AddFoodItem(foodItem);
                        rows++;
                    }
                    else
                    {
                        Console.WriteLine($"DEBUG: Could not find restaurant with ID: [{restaurantID}]");
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Error processing food item: {ex.Message}");
                }
            }
            Console.WriteLine($"{rows} Food Items loaded!");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading food items: {ex.Message}");
    }
}

Restaurant FindRestaurant(string id)
{
    foreach (Restaurant r in RestaurantList)
    {
        if (r.RestaurantId == id)
        {
            return r;
        }
    }
    return null;
}

// Basic Feature 2 : Anjushree
void LoadCustomers(string path)
{
    try
    {
        if (!File.Exists(path))
        {
            Console.WriteLine("customers.csv file not found!");
            return;
        }

        customers.Clear();
        customerByEmail.Clear();

        foreach (string line in File.ReadAllLines(path))
        {
            try
            {
                if (string.IsNullOrWhiteSpace(line)) continue;
                if (line.StartsWith("Name")) continue;

                string[] p = line.Split(',');
                if (p.Length < 2) continue;

                string name = p[0].Trim();
                string email = p[1].Trim();

                Customer c = new Customer(email, name);

                customers.Add(c);
                customerByEmail[email] = c;
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error processing customer line: {ex.Message}");
            }
        }

        Console.WriteLine($"{customers.Count} customers loaded!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading customers: {ex.Message}");
    }
}

void LoadOrders(string path)
{
    try
    {
        if (!File.Exists(path))
        {
            Console.WriteLine("orders.csv file not found!");
            return;
        }

        string[] lines = File.ReadAllLines(path);
        if (lines.Length <= 1)
        {
            Console.WriteLine("0 orders loaded!");
            return;
        }

        int count = 0;
        int itemCount = 0;

        // Skip header (line 0), process data lines
        for (int i = 1; i < lines.Length; i++)
        {
            try
            {
                string line = lines[i];
                if (string.IsNullOrWhiteSpace(line)) continue;

                // Use Regex to properly parse CSV with quoted fields
                // This handles: value1,value2,"quoted, value",value4
                var regex = new Regex(",(?=(?:[^\"]*\"[^\"]*\")*[^\"]*$)");
                string[] fields = regex.Split(line);

                if (fields.Length < 10)
                {
                    Console.WriteLine($"Line {i}: Not enough fields, skipping");
                    continue;
                }

                // Extract fields by position (based on header)
                // OrderId,CustomerEmail,RestaurantId,DeliveryDate,DeliveryTime,DeliveryAddress,CreatedDateTime,TotalAmount,Status,Items
                string orderIdStr = fields[0].Trim();
                string custEmail = fields[1].Trim();
                string restId = fields[2].Trim();
                string deliveryDate = fields[3].Trim();
                string deliveryTime = fields[4].Trim();
                string address = fields[5].Trim();
                string createdDT = fields[6].Trim();
                string totalStr = fields[7].Trim();
                string status = fields[8].Trim();
                string itemsStr = fields[9].Trim().Trim('"'); // Remove surrounding quotes

                // Parse order ID
                if (!int.TryParse(orderIdStr, out int orderId))
                {
                    Console.WriteLine($"Line {i}: Invalid order ID '{orderIdStr}', skipping");
                    continue;
                }

                // Check customer exists
                if (!customerByEmail.ContainsKey(custEmail))
                {
                    Console.WriteLine($"Order {orderId}: Customer '{custEmail}' not found, skipping");
                    continue;
                }

                // Check restaurant exists
                if (!restaurantById.ContainsKey(restId))
                {
                    Console.WriteLine($"Order {orderId}: Restaurant '{restId}' not found, skipping");
                    continue;
                }

                // Parse dates
                DateTime orderDateTime = DateTime.Now;
                if (!string.IsNullOrEmpty(createdDT))
                {
                    DateTime.TryParse(createdDT, out orderDateTime);
                }

                DateTime deliveryDateTime = DateTime.Now;
                if (!string.IsNullOrEmpty(deliveryDate) && !string.IsNullOrEmpty(deliveryTime))
                {
                    DateTime.TryParse($"{deliveryDate} {deliveryTime}", out deliveryDateTime);
                }

                // Parse total
                if (!double.TryParse(totalStr, out double total))
                {
                    total = 0;
                }

                // Create order
                Order order = new Order(orderId, orderDateTime, total, status, deliveryDateTime, address, "", false);
                order.Customer = customerByEmail[custEmail];
                order.Restaurant = restaurantById[restId];

                // Add to collections
                customerByEmail[custEmail].AddOrder(order);
                restaurantById[restId].Orders.Enqueue(order);
                orderRestaurantMap[orderId] = restId;

                if (orderId >= nextOrderId)
                {
                    nextOrderId = orderId + 1;
                }

                // Parse items: "Item1, qty1|Item2, qty2"
                if (!string.IsNullOrEmpty(itemsStr))
                {
                    string[] items = itemsStr.Split('|');

                    foreach (string item in items)
                    {
                        try
                        {
                            if (string.IsNullOrWhiteSpace(item)) continue;

                            // Split by comma: "Chicken Katsu Bento, 1"
                            int lastComma = item.LastIndexOf(',');
                            if (lastComma == -1) continue;

                            string itemName = item.Substring(0, lastComma).Trim();
                            string qtyStr = item.Substring(lastComma + 1).Trim();

                            if (!int.TryParse(qtyStr, out int qty) || qty < 1)
                            {
                                Console.WriteLine($"Order {orderId}: Invalid quantity '{qtyStr}' for '{itemName}'");
                                continue;
                            }

                            // Find the food item in the restaurant's menu
                            FoodItem foundItem = null;
                            foreach (Menu menu in order.Restaurant.Menus)
                            {
                                foundItem = menu.FoodItems.Find(f =>
                                    f.ItemName.Equals(itemName, StringComparison.OrdinalIgnoreCase));

                                if (foundItem != null) break;
                            }

                            if (foundItem != null)
                            {
                                OrderedFoodItem orderedItem = new OrderedFoodItem(
                                    foundItem.ItemName,
                                    foundItem.ItemDesc,
                                    foundItem.ItemPrice,
                                    "",
                                    qty
                                );

                                order.AddOrderedFoodItem(orderedItem);
                                itemCount++;
                            }
                            else
                            {
                                Console.WriteLine($"Order {orderId}: Food item '{itemName}' not found in {restId}");
                            }
                        }
                        catch (Exception ex)
                        {
                            Console.WriteLine($"Order {orderId}: Error parsing item '{item}': {ex.Message}");
                        }
                    }
                }

                count++;
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Line {i}: Error - {ex.Message}");
            }
        }

        Console.WriteLine($"{count} orders loaded with {itemCount} food items!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading orders: {ex.Message}");
    }
}


// Basic Feature 3: Anjushree 
void ListAllRestaurantsAndMenuItems()
{
    try
    {
        Console.WriteLine("\nAll Restaurants and Menu Items");
        Console.WriteLine("==============================");

        foreach (Restaurant r in RestaurantList)
        {
            Console.WriteLine($"Restaurant: {r.RestaurantName} ({r.RestaurantId})");

            if (r.Menus.Count == 0)
            {
                Console.WriteLine("  (No menu)");
                Console.WriteLine();
                continue;
            }

            foreach (Menu m in r.Menus)
            {
                Console.WriteLine($"  {m.ToString()}");

                if (m.FoodItems.Count == 0)
                {
                    Console.WriteLine("    (No food items)");
                }
                else
                {
                    foreach (FoodItem fi in m.FoodItems)
                    {
                        Console.WriteLine($"    - {fi}");
                    }
                }
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error displaying restaurants and menus: {ex.Message}");
    }
}

// basic feature 4 : Gui Ru
void ListAllOrders()
{
    try
    {
        Console.WriteLine("\nAll Orders");
        Console.WriteLine("==========");
        Console.WriteLine($"{"Order ID",-9}{"Customer",-15}{"Restaurant",-15}{"Delivery Date/Time",-20}{"Amount",-8}{"Status",-12}");
        Console.WriteLine(new string('-', 90));

        int orderCount = 0;
        foreach (Customer customer in customers)
        {
            foreach (Order order in customer.OrderList)
            {
                Console.WriteLine(order.ToString());
                orderCount++;
            }
        }

        if (orderCount == 0)
        {
            Console.WriteLine("No orders found.");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error listing orders: {ex.Message}");
    }
}

// basic feature 5 : Anjushree
void CreateNewOrder()
{
    try
    {
        Console.WriteLine("\n=== Create New Order ===");

        // Get customer
        Console.Write("Enter Customer Email: ");
        string custEmail = Console.ReadLine().Trim();
        if (!customerByEmail.ContainsKey(custEmail))
        {
            Console.WriteLine("Customer not found!");
            return;
        }
        Customer c = customerByEmail[custEmail];

        // List restaurants
        Console.WriteLine("\nAvailable Restaurants:");
        for (int i = 0; i < RestaurantList.Count; i++)
        {
            Console.WriteLine($"{i + 1}. {RestaurantList[i].RestaurantName} ({RestaurantList[i].RestaurantId})");
        }

        Console.Write("Select a restaurant (enter number): ");
        if (!int.TryParse(Console.ReadLine(), out int restChoice) ||
            restChoice < 1 || restChoice > RestaurantList.Count)
        {
            Console.WriteLine("Invalid selection!");
            return;
        }

        Restaurant selectedRest = RestaurantList[restChoice - 1];

        if (selectedRest.Menus.Count == 0)
        {
            Console.WriteLine("This restaurant has no menu!");
            return;
        }

        Menu menu = selectedRest.Menus[0];

        // Select food items
        List<OrderedFoodItem> orderedItems = SelectOrderedFoodItems(menu);
        if (orderedItems.Count == 0)
        {
            Console.WriteLine("No items selected. Order cancelled.");
            return;
        }

        // Get delivery details
        Console.Write("Enter Delivery Address: ");
        string address = Console.ReadLine().Trim();
        if (string.IsNullOrWhiteSpace(address))
        {
            Console.WriteLine("Address cannot be empty!");
            return;
        }

        DateTime deliveryDT = PromptDateTime("Enter Delivery Date/Time (dd/MM/yyyy HH:mm): ");

        Console.Write("Payment Method (Cash/Card/Online): ");
        string payMethod = Console.ReadLine().Trim();

        // Create order
        Order newOrder = new Order(
            nextOrderId++,
            DateTime.Now,
            0,
            "Pending",
            deliveryDT,
            address,
            payMethod,
            false
        );

        newOrder.Customer = c;
        newOrder.Restaurant = selectedRest;

        foreach (var item in orderedItems)
        {
            newOrder.AddOrderedFoodItem(item);
        }

        newOrder.OrderTotal = newOrder.CalculateOrderTotal();

        c.AddOrder(newOrder);
        selectedRest.Orders.Enqueue(newOrder);

        Console.WriteLine($"\nOrder created successfully! Order ID: {newOrder.OrderId}");
        Console.WriteLine($"Total: ${newOrder.OrderTotal:F2}");

        // Optionally save to CSV
        SaveOrderToCsv(newOrder, custEmail, selectedRest.RestaurantId);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error creating order: {ex.Message}");
    }
}

List<OrderedFoodItem> SelectOrderedFoodItems(Menu menu)
{
    List<OrderedFoodItem> items = new List<OrderedFoodItem>();

    try
    {
        while (true)
        {
            Console.WriteLine("\nMenu Items:");
            menu.DisplayFoodItems();

            Console.Write("\nSelect item number (0 to finish): ");
            if (!int.TryParse(Console.ReadLine(), out int choice))
            {
                Console.WriteLine("Invalid input!");
                continue;
            }

            if (choice == 0) break;

            if (choice < 1 || choice > menu.FoodItems.Count)
            {
                Console.WriteLine("Invalid selection!");
                continue;
            }

            FoodItem selectedItem = menu.FoodItems[choice - 1];

            Console.Write("Enter quantity: ");
            if (!int.TryParse(Console.ReadLine(), out int qty) || qty < 1)
            {
                Console.WriteLine("Invalid quantity!");
                continue;
            }

            OrderedFoodItem ofi = new OrderedFoodItem(
                selectedItem.ItemName,
                selectedItem.ItemDesc,
                selectedItem.ItemPrice,
                "",
                qty
            );

            items.Add(ofi);
            Console.WriteLine($"Added {qty}x {selectedItem.ItemName}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error selecting items: {ex.Message}");
    }

    return items;
}

DateTime PromptDateTime(string message)
{
    while (true)
    {
        try
        {
            Console.Write(message);
            string input = Console.ReadLine();
            if (DateTime.TryParseExact(input, "dd/MM/yyyy HH:mm", null,
                System.Globalization.DateTimeStyles.None, out DateTime result))
            {
                return result;
            }
            Console.WriteLine("Invalid format. Use dd/MM/yyyy HH:mm (e.g., 15/02/2026 14:30)");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error parsing date: {ex.Message}");
        }
    }
}

string PromptNonEmpty(string message)
{
    while (true)
    {
        Console.Write(message);
        string input = Console.ReadLine()?.Trim();
        if (!string.IsNullOrWhiteSpace(input))
            return input;
        Console.WriteLine("Input cannot be empty. Try again.");
    }
}

void SaveOrderToCsv(Order o, string custEmail, string restId)
{
    try
    {
        // Build items string
        List<string> itemParts = new List<string>();
        foreach (var ofi in o.OrderedFoodItem)
        {
            itemParts.Add($"{ofi.ItemName}, {ofi.QtyOrdered}");
        }
        string itemsStr = string.Join("|", itemParts);

        string line = $"{o.OrderId},{custEmail},{restId}," +
                      $"{o.DeliveryDateTime:dd/MM/yyyy},{o.DeliveryDateTime:HH:mm}," +
                      $"{o.DeliveryAddress},{o.OrderDateTime:dd/MM/yyyy HH:mm}," +
                      $"{o.OrderTotal:F2},{o.OrderStatus}," +
                      $"\"{itemsStr}\"";

        File.AppendAllText(ordersFilePath, line + Environment.NewLine);
        Console.WriteLine("Order saved to CSV file.");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error saving order to CSV: {ex.Message}");
    }
}

// basic feature 6 : Gui Ru
void ProcessOrder()
{
    try
    {
        Console.WriteLine("\nProcess Order");
        Console.WriteLine("=============");

        Console.Write("Enter Customer Email: ");
        string email = Console.ReadLine().Trim();

        Customer customer = FindCustomer(email);
        if (customer == null)
        {
            Console.WriteLine("Customer not found!");
            return;
        }

        List<Order> pendingOrders = customer.OrderList
            .Where(o => o.OrderStatus.Equals("Pending", StringComparison.OrdinalIgnoreCase))
            .ToList();

        if (pendingOrders.Count == 0)
        {
            Console.WriteLine("No pending orders for this customer.");
            return;
        }

        Console.WriteLine("\nPending Orders:");
        Console.WriteLine($"{"Order ID",-10}{"Restaurant",-20}{"Delivery Time",-20}{"Total",-10}");
        Console.WriteLine(new string('-', 60));

        foreach (Order order in pendingOrders)
        {
            Console.WriteLine($"{order.OrderId,-10}{order.Restaurant?.RestaurantName,-20}" +
                            $"{order.DeliveryDateTime,-20:dd/MM/yyyy HH:mm}${order.OrderTotal,-10:F2}");
        }

        Console.Write("\nEnter Order ID to process: ");
        if (!int.TryParse(Console.ReadLine(), out int orderID))
        {
            Console.WriteLine("Invalid Order ID!");
            return;
        }

        Order orderToProcess = FindOrder(customer, orderID);
        if (orderToProcess == null)
        {
            Console.WriteLine("Order not found!");
            return;
        }

        if (orderToProcess.OrderStatus != "Pending")
        {
            Console.WriteLine($"Cannot process order. Current status: {orderToProcess.OrderStatus}");
            return;
        }

        Console.WriteLine($"\nOrder Details:");
        Console.WriteLine($"Restaurant: {orderToProcess.Restaurant?.RestaurantName}");
        orderToProcess.DisplayOrderedFoodItem();

        Console.WriteLine("\nConfirm processing this order? (Y/N): ");
        string confirm = Console.ReadLine().Trim().ToUpper();

        if (confirm == "Y")
        {
            orderToProcess.OrderStatus = "Preparing";
            Console.WriteLine($"Order {orderID} is now being prepared!");

            UpdateOrderRowInCsv(ordersFilePath, orderToProcess, email, orderToProcess.Restaurant.RestaurantId);
        }
        else
        {
            Console.WriteLine("Processing cancelled.");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error processing order: {ex.Message}");
    }
}

// basic feature 7 : Gui Ru
void ModifyExistingOrder()
{
    try
    {
        Console.WriteLine("\nModify Order");
        Console.WriteLine("============");

        Console.Write("Enter Customer Email: ");
        string email = Console.ReadLine().Trim();

        Customer customer = FindCustomer(email);
        if (customer == null)
        {
            Console.WriteLine("Customer not found!");
            return;
        }

        List<Order> pendingOrders = GetPendingOrders(customer);
        if (pendingOrders.Count == 0)
        {
            Console.WriteLine("No pending orders to modify.");
            return;
        }

        Console.WriteLine("\nPending Orders:");
        foreach (Order o in pendingOrders)
        {
            Console.WriteLine($"Order ID: {o.OrderId}, Restaurant: {o.Restaurant?.RestaurantName}, " +
                            $"Total: ${o.OrderTotal:F2}");
        }

        Console.Write("\nEnter Order ID to modify: ");
        if (!int.TryParse(Console.ReadLine(), out int orderID))
        {
            Console.WriteLine("Invalid Order ID!");
            return;
        }

        Order orderToModify = FindOrder(customer, orderID);
        if (orderToModify == null || orderToModify.OrderStatus != "Pending")
        {
            Console.WriteLine("Order not found or cannot be modified!");
            return;
        }

        string restId = orderRestaurantMap.ContainsKey(orderID) ? orderRestaurantMap[orderID] : "";
        Menu menu = orderToModify.Restaurant?.Menus.FirstOrDefault();

        if (menu == null)
        {
            Console.WriteLine("Restaurant menu not available!");
            return;
        }

        string choice;
        while (true)
        {
            Console.WriteLine("\nWhat would you like to modify?");
            Console.WriteLine("1. Change food items");
            Console.WriteLine("2. Change delivery address");
            Console.WriteLine("3. Change delivery date/time");
            Console.Write("Enter choice (1-3): ");
            choice = Console.ReadLine()?.Trim();

            if (choice == "1" || choice == "2" || choice == "3") break;
            Console.WriteLine("Invalid choice. Enter 1, 2, or 3.");
        }

        if (choice == "1")
        {
            Console.WriteLine("\nReselect items for this order:");
            List<OrderedFoodItem> newItems = SelectOrderedFoodItems(menu);

            orderToModify.OrderedFoodItem.Clear();
            foreach (var ofi in newItems)
                orderToModify.AddOrderedFoodItem(ofi);

            orderToModify.OrderTotal = orderToModify.CalculateOrderTotal();

            UpdateOrderRowInCsv(ordersFilePath, orderToModify, customer.EmailAddress, restId);

            Console.WriteLine("Order items updated successfully!");
            Console.WriteLine($"New total: ${orderToModify.OrderTotal:F2}");
        }
        else if (choice == "2")
        {
            string newAddress = PromptNonEmpty("Enter new delivery address: ");
            orderToModify.DeliveryAddress = newAddress;

            UpdateOrderRowInCsv(ordersFilePath, orderToModify, customer.EmailAddress, restId);
            Console.WriteLine("Delivery address updated successfully!");
        }
        else // choice == "3"
        {
            DateTime newDT = PromptDateTime("Enter new Delivery Date/Time (dd/MM/yyyy HH:mm): ");
            orderToModify.DeliveryDateTime = newDT;

            UpdateOrderRowInCsv(ordersFilePath, orderToModify, customer.EmailAddress, restId);
            Console.WriteLine("Delivery date/time updated successfully!");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error modifying order: {ex.Message}");
    }
}

List<Order> GetPendingOrders(Customer c)
{
    List<Order> result = new();
    foreach (Order o in c.OrderList)
        if (o.OrderStatus == "Pending")
            result.Add(o);
    return result;
}

Order FindOrder(Customer c, int orderId)
{
    foreach (Order o in c.OrderList)
        if (o.OrderId == orderId)
            return o;
    return null;
}

int PromptInt(string message)
{
    while (true)
    {
        try
        {
            Console.Write(message);
            if (int.TryParse(Console.ReadLine(), out int value))
                return value;
            Console.WriteLine("Invalid number. Try again.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    }
}

void UpdateOrderRowInCsv(string path, Order o, string custEmail, string restId)
{
    try
    {
        if (!File.Exists(path))
        {
            Console.WriteLine("Orders file not found. Cannot update CSV.");
            return;
        }

        string[] lines = File.ReadAllLines(path);
        if (lines.Length == 0) return;

        // rebuild itemsStr from OrderedFoodItem list
        List<string> itemParts = new();
        foreach (var ofi in o.OrderedFoodItem)
            itemParts.Add($"{ofi.ItemName}, {ofi.QtyOrdered}");
        string itemsStr = string.Join("|", itemParts);

        for (int i = 1; i < lines.Length; i++)
        {
            if (string.IsNullOrWhiteSpace(lines[i])) continue;

            string[] parts = lines[i].Split(',');
            if (parts.Length == 0) continue;

            if (int.TryParse(parts[0].Trim(), out int id) && id == o.OrderId)
            {
                string newLine =
                    $"{o.OrderId},{custEmail},{restId}," +
                    $"{o.DeliveryDateTime:dd/MM/yyyy},{o.DeliveryDateTime:HH:mm}," +
                    $"{o.DeliveryAddress},{o.OrderDateTime:dd/MM/yyyy HH:mm}," +
                    $"{o.OrderTotal:F2},{o.OrderStatus}," +
                    $"\"{itemsStr}\"";

                lines[i] = newLine;
                File.WriteAllLines(path, lines);
                return;
            }
        }

        Console.WriteLine("Order ID not found in CSV. No update made.");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error updating CSV: {ex.Message}");
    }
}

// basic feature 8 : Gui Ru 
void DeleteOrder()
{
    try
    {
        Console.WriteLine("\nDelete Order");
        Console.WriteLine("============");

        Console.Write("Enter Customer Email: ");
        string email = Console.ReadLine().Trim();

        Customer customer = FindCustomer(email);
        if (customer == null)
        {
            Console.WriteLine("Customer not found!");
            return;
        }

        List<Order> pendingOrders = customer.OrderList
            .Where(o => o.OrderStatus.Equals("Pending", StringComparison.OrdinalIgnoreCase))
            .ToList();

        if (pendingOrders.Count == 0)
        {
            Console.WriteLine("No pending orders for this customer.");
            return;
        }

        Console.WriteLine("Pending Orders:");
        foreach (Order order in pendingOrders)
        {
            Console.WriteLine($"Order ID: {order.OrderId}");
        }

        Console.Write("Enter Order ID: ");
        if (!int.TryParse(Console.ReadLine(), out int orderID))
        {
            Console.WriteLine("Invalid Order ID format!");
            return;
        }

        Order orderToDelete = FindOrderID(orderID);
        if (orderToDelete == null)
        {
            Console.WriteLine("Order not found!");
            return;
        }

        if (orderToDelete.OrderStatus != "Pending")
        {
            Console.WriteLine($"Cannot delete order. Current status: {orderToDelete.OrderStatus}");
            Console.WriteLine("Only pending orders can be deleted.");
            return;
        }

        // Display order details
        Console.WriteLine($"\nCustomer: {customer.CustomerName}");
        Console.WriteLine("Ordered Items:");
        int itemNum = 1;
        foreach (var item in orderToDelete.OrderedFoodItem)
        {
            Console.WriteLine($"{itemNum}. {item.ItemName} - ${item.ItemPrice:F2} x {item.QtyOrdered}");
            itemNum++;
        }
        Console.WriteLine($"Delivery date/time: {orderToDelete.DeliveryDateTime:dd/MM/yyyy HH:mm}");
        Console.WriteLine($"Total Amount: ${orderToDelete.CalculateOrderTotal():F2}");
        Console.WriteLine($"Order Status: {orderToDelete.OrderStatus}");

        Console.Write("\nConfirm deletion? [Y/N]: ");
        string confirm = Console.ReadLine().Trim().ToUpper();

        if (confirm == "Y")
        {
            orderToDelete.OrderStatus = "Cancelled";
            RefundStack.Push(orderToDelete);
            Console.WriteLine($"Order {orderID} cancelled. Refund of ${orderToDelete.CalculateOrderTotal():F2} processed.");

            UpdateOrderRowInCsv(ordersFilePath, orderToDelete, email, orderToDelete.Restaurant.RestaurantId);
        }
        else
        {
            Console.WriteLine("Deletion cancelled.");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error deleting order: {ex.Message}");
    }
}

Customer FindCustomer(string email)
{
    foreach (Customer customer in customers)
    {
        if (customer.EmailAddress.Equals(email, StringComparison.OrdinalIgnoreCase))
        {
            return customer;
        }
    }
    return null;
}

Order FindOrderID(int id)
{
    foreach (Customer c in customers)
    {
        foreach (Order o in c.OrderList)
        {
            if (o.OrderId == id) return o;
        }
    }
    return null;
}

void SaveQueueAndStack()
{
    try
    {
        // Save queue data
        using (StreamWriter sw = new StreamWriter("queue.csv"))
        {
            sw.WriteLine("RestaurantID,OrderID,CustomerEmail,Status");
            foreach (Restaurant restaurant in RestaurantList)
            {
                foreach (Order order in restaurant.Orders)
                {
                    sw.WriteLine($"{restaurant.RestaurantId},{order.OrderId},{order.Customer.EmailAddress},{order.OrderStatus}");
                }
            }
        }

        // Save stack data
        using (StreamWriter sw = new StreamWriter("stack.csv"))
        {
            sw.WriteLine("OrderID,CustomerEmail,RestaurantID,TotalAmount,Status");
            foreach (Order order in RefundStack)
            {
                sw.WriteLine($"{order.OrderId},{order.Customer.EmailAddress},{order.Restaurant.RestaurantId},{order.CalculateOrderTotal():F2},{order.OrderStatus}");
            }
        }

        Console.WriteLine("\nQueue and stack data saved successfully!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error saving data: {ex.Message}");
    }
}

