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
using System.Globalization;
using System.IO;
using System.Linq;



List<Restaurant> RestaurantList = new List<Restaurant>();
Dictionary<string, Restaurant> restaurantById = new();

List<Customer> customers = new();
Dictionary<string, Customer> customerByEmail = new();

Dictionary<int, string> orderRestaurantMap = new();
Stack<Order> refundStack = new();

int nextOrderId = 1;
string ordersFilePath = "orders - Copy.csv";


Console.WriteLine("Welcome to the Gruberoo Food Delivery System");
LoadRestaurants();
LoadMenuItems();
LoadCustomers("customers.csv");
LoadOrders("orders - Copy.csv");

while (true)
{
    DisplayMainMenu();
    Console.Write("Enter your choice: ");
    int option = Convert.ToInt32(Console.ReadLine());


    if (option == 1) ListAllRestaurantsAndMenuItems();
    else if (option == 2) ListAllOrders();
    else if (option == 3) CreateNewOrder();
    else if (option == 4) ProcessOrder();
    else if (option == 5) ModifyExistingOrder();
    else if (option == 6) DeleteOrder();
    else if (option == 0) break;
}
// Main menu loop

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



// Basic Feature 1: Gui Ru 
void LoadAllRestaurantsAndMenuItems()
{
    LoadRestaurants();
    LoadFoodItems();
}

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
                string[] data = line.Split(',');
                string restaurantID = data[0].Trim();
                string name = data[1].Trim();
                string email = data[2].Trim();

                Restaurant r = new Restaurant(restaurantID, name, email);
                RestaurantList.Add(r);

                count++;
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
                    string[] data = line.Split(',');
                    string restaurantID = data[0].Trim();
                    string itemName = data[1].Trim();
                    string desc = data[2].Trim();
                    double price = Convert.ToDouble(data[3].Trim());

                    Restaurant restaurant = FindRestaurant(restaurantID);

                    if (restaurant != null)
                    {
                        // Create food item with empty customise field
                        FoodItem foodItem = new FoodItem(itemName, desc, price, "");

                        // Check if restaurant has a menu, if not create one
                        if (restaurant.Menus.Count == 0)
                        {
                            restaurant.AddMenu(new Menu("M001", "Main Menu"));  //each menu for each restaurant. same name for diff restaurant
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
            return r; // Success: Found the match!
        }
    }
    return null; // Failure: ID doesn't exist in our list
}

// Basic Feature 2 : Anjushree
void LoadCustomers(string path)
        {
            customers.Clear();
            customerByEmail.Clear();

            foreach (string line in File.ReadAllLines(path))
            {
                if (string.IsNullOrWhiteSpace(line)) continue;
                if (line.StartsWith("Name")) continue;

                string[] p = line.Split(',');
                if (p.Length < 2) continue;

                string name = p[0].Trim();
                string email = p[1].Trim();

                // Customer constructor: Customer(string ea, string cn)
                Customer c = new Customer(email, name);

                customers.Add(c);
                customerByEmail[email] = c;
            }

            Console.WriteLine($"{customers.Count} customers loaded!");
        }

void LoadOrders(string path)
        {
            string[] lines = File.ReadAllLines(path);
            if (lines.Length <= 1)
            {
                Console.WriteLine("0 orders loaded!");
                return;
            }

            // header -> index map
            string[] headers = lines[0].Split(',');
            Dictionary<string, int> idx = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase);
            for (int i = 0; i < headers.Length; i++)
                idx[headers[i].Trim()] = i;

            string Get(string[] parts, params string[] names)
            {
                foreach (string n in names)
                {
                    if (idx.ContainsKey(n))
                    {
                        int k = idx[n];
                        if (k >= 0 && k < parts.Length) return parts[k].Trim();
                    }
                }
                return "";
            }

            int count = 0;

            for (int i = 1; i < lines.Length; i++)
            {
                if (string.IsNullOrWhiteSpace(lines[i])) continue;
                string[] p = lines[i].Split(',');

                string orderIdStr = Get(p, "OrderID", "Order Id", "Order ID");
                string custEmail = Get(p, "CustomerEmail", "Customer Email", "Email");
                string restId = Get(p, "RestaurantID", "Restaurant Id", "Restaurant ID");
                string status = Get(p, "Status");
                string totalStr = Get(p, "TotalAmount", "Total Amount", "Amount");

                string orderDTStr = Get(p, "OrderDateTime", "Order Date/Time", "Order Date Time");
                string delDTStr = Get(p, "DeliveryDateTime", "Delivery Date/Time", "Delivery Date Time");
                string address = Get(p, "DeliveryAddress", "Address");
                string payMethod = Get(p, "PaymentMethod", "Payment Method");
                string paidStr = Get(p, "Paid", "OrderPaid");

                if (!int.TryParse(orderIdStr, out int orderId)) continue;
                if (!double.TryParse(totalStr, out double total)) total = 0;

                if (!customerByEmail.ContainsKey(custEmail)) continue;
                if (!restaurantById.ContainsKey(restId)) continue;

                DateTime orderDT = DateTime.Now;
                DateTime deliveryDT = DateTime.Now;
                if (!string.IsNullOrEmpty(orderDTStr)) DateTime.TryParse(orderDTStr, out orderDT);
                if (!string.IsNullOrEmpty(delDTStr)) DateTime.TryParse(delDTStr, out deliveryDT);

                bool paid = false;
                if (!string.IsNullOrEmpty(paidStr))
                {
                    if (paidStr.Equals("Y", StringComparison.OrdinalIgnoreCase)) paid = true;
                    else if (paidStr.Equals("N", StringComparison.OrdinalIgnoreCase)) paid = false;
                    else bool.TryParse(paidStr, out paid);
                }

                // Order constructor:
                // Order(int oi, DateTime odt, double ot, string os, DateTime ddt, string da, string opm, bool op)
                Order o = new Order(orderId, orderDT, total, status, deliveryDT, address, payMethod, paid);

                // ✅ Feature 2 required links:
                customerByEmail[custEmail].AddOrder(o);
                restaurantById[restId].Orders.Enqueue(o);

                count++;

                orderRestaurantMap[orderId] = restId;
                if (orderId >= nextOrderId) nextOrderId = orderId + 1;

            }

            Console.WriteLine($"{count} orders loaded!");
        }



        // Basic Feature 3: Anjushree 
        void ListAllRestaurantsAndMenuItems()
        {
            Console.WriteLine("\nAll Restaurants and Menu Items");
            Console.WriteLine("==============================");

            foreach (Restaurant r in restaurants)
            {
                Console.WriteLine($"Restaurant: {r.RestaurantName} ({r.RestaurantId})");

                foreach (Menu m in r.Menus)
                {
                    foreach (FoodItem fi in m.FoodItems)
                    {
                        Console.WriteLine($" - {fi}");
                    }
                }

                Console.WriteLine();
            }
        }



        // Basic Feature 4: Gui Ru 
   void ListAllOrders()
{
    Console.WriteLine("\nAll Orders");
    Console.WriteLine("==========");
    Console.WriteLine($"{"Order ID",-9}{"Customer",-15}{"Restaurant",-15}{"Delivery Date/Time",-20}{"Amount",-8}{"Status",-12}");
    Console.WriteLine(new string('-', 90));

    foreach (Customer customer in CustomerList)
    {
        foreach (Order order in customer.OrderList)
        {
            Console.WriteLine(order.ToString());
        }
    }
}
        
        // Basic Feature 5: Anjushree 
void CreateNewOrder()
{
    Console.WriteLine("\nCreate New Order");
    Console.WriteLine("================");

    Customer customer = PromptCustomerByEmail();
    Restaurant restaurant = PromptRestaurantById();
    string restId = restaurant.RestaurantId;

    if (restaurant.Menus == null || restaurant.Menus.Count == 0)
    {
        Console.WriteLine("Restaurant has no menu.");
        return;
    }

    Menu menu = restaurant.Menus[0];

    if (menu.FoodItems == null || menu.FoodItems.Count == 0)
    {
        Console.WriteLine("This restaurant menu has no food items.");
        return;
    }

    DateTime deliveryDT = PromptDateTime("Enter Delivery Date/Time (dd/MM/yyyy HH:mm): ");
    string address = PromptNonEmpty("Enter Delivery Address: ");

    Dictionary<FoodItem, int> selected = SelectFoodItemsFromMenu(menu);
    string payMethod = PromptPaymentMethod();

    int orderId = nextOrderId++;
    DateTime orderDT = DateTime.Now;

    Order o = new Order(orderId, orderDT, 0, "Pending", deliveryDT, address, payMethod, true);

    //  Link first
    o.Customer = customer;
    o.Restaurant = restaurant;

    //  Add ordered items
    foreach (var kvp in selected)
    {
        FoodItem fi = kvp.Key;
        int qty = kvp.Value;

        OrderedFoodItem ofi = new OrderedFoodItem(
            fi.ItemName,
            fi.ItemDesc,
            fi.ItemPrice,
            fi.Customise,
            qty
        );

        o.AddOrderedFoodItem(ofi);
    }

    //  Calculate total (includes $5 delivery fee, and SpecialOffer discount if set)
    o.OrderTotal = o.CalculateOrderTotal();

    //  Store in system
    customer.AddOrder(o);
    restaurantById[restId].Orders.Enqueue(o);
    orderRestaurantMap[orderId] = restId;

    //  Save to CSV (same format as loader uses: ItemName:Qty;ItemName:Qty)
    AppendOrderToCsv(ordersFilePath, o, customer.EmailAddress, restId);

    Console.WriteLine($"\nOrder created! Order ID: {o.OrderId}");
    Console.WriteLine($"Total (incl $5 delivery): ${o.OrderTotal:F2}");
}

Customer PromptCustomerByEmail()
{
    while (true)
    {
        Console.Write("Enter Customer Email: ");
        string email = Console.ReadLine().Trim();

        if (customerByEmail.ContainsKey(email))
            return customerByEmail[email];

        Console.WriteLine("Invalid customer email. Try again.");
    }
}

Restaurant PromptRestaurantById()
{
    while (true)
    {
        Console.Write("Enter Restaurant ID: ");
        string id = Console.ReadLine().Trim();

        if (restaurantById.ContainsKey(id))
            return restaurantById[id];

        Console.WriteLine("Invalid restaurant ID. Try again.");
    }
}

string PromptNonEmpty(string msg)
{
    while (true)
    {
        Console.Write(msg);
        string s = Console.ReadLine();

        if (!string.IsNullOrWhiteSpace(s))
            return s.Trim();

        Console.WriteLine("Input cannot be empty.");
    }
}

DateTime PromptDateTime(string msg)
{
    while (true)
    {
        Console.Write(msg);
        string s = Console.ReadLine();

        if (DateTime.TryParseExact(
            s,
            "dd/MM/yyyy HH:mm",
            CultureInfo.InvariantCulture,
            DateTimeStyles.None,
            out DateTime dt))
            return dt;

        Console.WriteLine("Invalid format. Use dd/MM/yyyy HH:mm");
    }
}

string PromptPaymentMethod()
{
    while (true)
    {
        Console.Write("Payment Method [CC/PP/CD]: ");
        string pm = Console.ReadLine().Trim().ToUpper();

        if (pm == "CC" || pm == "PP" || pm == "CD")
            return pm;

        Console.WriteLine("Invalid method. Use CC, PP, or CD.");
    }
}

Dictionary<FoodItem, int> SelectFoodItemsFromMenu(Menu menu)
{
    Dictionary<FoodItem, int> selected = new Dictionary<FoodItem, int>();

    while (true)
    {
        Console.WriteLine("\nMenu Items:");
        for (int i = 0; i < menu.FoodItems.Count; i++)
            Console.WriteLine($"{i + 1}. {menu.FoodItems[i]}");

        Console.Write("Item number (0 to finish): ");
        if (!int.TryParse(Console.ReadLine(), out int n))
        {
            Console.WriteLine("Invalid number.");
            continue;
        }

        if (n == 0)
        {
            if (selected.Count == 0)
            {
                Console.WriteLine("Select at least one item.");
                continue;
            }
            break;
        }

        if (n < 1 || n > menu.FoodItems.Count)
        {
            Console.WriteLine("Invalid item number.");
            continue;
        }

        Console.Write("Quantity: ");
        if (!int.TryParse(Console.ReadLine(), out int q) || q <= 0)
        {
            Console.WriteLine("Invalid quantity.");
            continue;
        }

        FoodItem fi = menu.FoodItems[n - 1];
        if (selected.ContainsKey(fi)) selected[fi] += q;
        else selected[fi] = q;
    }

    return selected;
}

void AppendOrderToCsv(string path, Order o, string custEmail, string restId)
{
    List<string> parts = new List<string>();
    foreach (OrderedFoodItem ofi in o.OrderedFoodItem)
        parts.Add($"{ofi.ItemName}:{ofi.QtyOrdered}");

    string itemsStr = string.Join(";", parts);

    string line =
        $"{o.OrderId},{custEmail},{restId}," +
        $"{o.OrderDateTime:dd/MM/yyyy HH:mm}," +
        $"{o.DeliveryDateTime:dd/MM/yyyy HH:mm}," +
        $"{o.DeliveryAddress},{itemsStr}," +
        $"{o.OrderTotal:F2},{o.OrderStatus}," +
        $"{o.OrderPaymentMethod}";

    if (!File.Exists(path))
    {
        string header = "OrderID,CustomerEmail,RestaurantID,OrderDateTime,DeliveryDateTime,DeliveryAddress,Items,TotalAmount,Status,PaymentMethod";
        File.WriteAllText(path, header + Environment.NewLine + line);
    }
    else
    {
        File.AppendAllText(path, Environment.NewLine + line);
    }
}

List<OrderedFoodItem> SelectOrderedFoodItems(Menu menu)
{
    List<OrderedFoodItem> selected = new List<OrderedFoodItem>();

    while (true)
    {
        Console.WriteLine("\nMenu Items:");
        for (int i = 0; i < menu.FoodItems.Count; i++)
        {
            Console.WriteLine($"{i + 1}. {menu.FoodItems[i]}");
        }

        Console.Write("Item number (0 to finish): ");
        if (!int.TryParse(Console.ReadLine(), out int choice))
        {
            Console.WriteLine("Invalid input.");
            continue;
        }

        if (choice == 0)
        {
            if (selected.Count == 0)
            {
                Console.WriteLine("You must select at least one item.");
                continue;
            }
            break;
        }

        if (choice < 1 || choice > menu.FoodItems.Count)
        {
            Console.WriteLine("Invalid item number.");
            continue;
        }

        Console.Write("Quantity: ");
        if (!int.TryParse(Console.ReadLine(), out int qty) || qty <= 0)
        {
            Console.WriteLine("Invalid quantity.");
            continue;
        }

        FoodItem fi = menu.FoodItems[choice - 1];

        OrderedFoodItem ofi = new OrderedFoodItem(
            fi.ItemName,
            fi.ItemDesc,
            fi.ItemPrice,
            fi.Customise,
            qty
        );

        selected.Add(ofi);
    }

    return selected;
}

        
        
        // Basic Feature 6: Gui Ru 

        void ProcessOrder()
{
    Console.WriteLine("\nProcess Order");
    Console.WriteLine("=============");
    Console.Write("Enter Restaurant ID: ");
    string restaurantID = Console.ReadLine().Trim();

    // finding restaurant
    Restaurant restaurant = FindRestaurant(restaurantID);
    if (restaurant == null) 
    {
        Console.WriteLine("Restaurant not found!"); 
        return; 
    }
    if (restaurant.Orders.Count == 0) 
    {
        Console.WriteLine("No orders for this restaurant."); 
        return; 
    }

    Queue<Order> tempQueue = new Queue<Order>();

    while (restaurant.Orders.Count > 0)
    {
        Order order = restaurant.Orders.Dequeue();

        // Display info matching Figure 8 requirements
        // Display order number
        Console.WriteLine($"\nOrder {order.OrderId}:");
        // Display Customer Name
        Console.WriteLine($"Customer: {order.Customer.CustomerName}");
        // Display ordered food item
        Console.WriteLine("Ordered Items:");
        int itemNum = 1;
        foreach (FoodItem entry in order.OrderedFoodItem)
        {
            Console.WriteLine($"{itemNum}. {entry.ItemName} - {entry.ItemPrice}");
            itemNum++;
        }
        // Display delivery 
        Console.WriteLine($"Delivery date/time: {order.DeliveryDateTime:dd/MM/yyyy HH:mm}");

        // Display Total Amount 
        Console.WriteLine($"Total Amount: ${order.CalculateOrderTotal():F2}");

        // Display Order Status
        Console.WriteLine($"Order Status: {order.OrderStatus}");


        Console.Write("\n[C]onfirm / [R]eject / [S]kip / [D]eliver: ");
        string action = Console.ReadLine().Trim().ToUpper();

        if (action == "C")
        {
            if (order.OrderStatus == "Pending")
            {
                order.OrderStatus = "Preparing"; // Updates status to Preparing
                Console.WriteLine($"Order {order.OrderId} confirmed.");
            }
            tempQueue.Enqueue(order);
            break;
        }
        else if (action == "R")
        {
            if (order.OrderStatus == "Pending")
            {
                order.OrderStatus = "Rejected";
                RefundStack.Push(order); // Add to Refund Stack (LIFO)
                Console.WriteLine($"Order {order.OrderId} rejected. Refund processed.");
            }
            break;
        }
        else if (action == "S")
        {
            if (order.OrderStatus == "Preparing")
            {
                order.OrderStatus = "Delivered";
                Console.WriteLine($"Order {order.OrderId} delivered.");
            }
            tempQueue.Enqueue(order);
            break;
        }
        else
        {
            tempQueue.Enqueue(order); // Keep in queue if skipped
        }          
    }
    // Restore the restaurant's queue
    foreach (var o in tempQueue) { restaurant.Orders.Enqueue(o); }
}

       // Basic Feature 7: Anjushree

void ModifyExistingOrder()
{
    Console.WriteLine("\nModify Existing Order");
    Console.WriteLine("=====================");

    // 1) Pick a valid customer
    Customer customer = PromptCustomerByEmail();

    // 2) Customer must have at least 1 pending order
    List<Order> pending = GetPendingOrders(customer);
    while (pending.Count == 0)
    {
        Console.WriteLine("No pending orders for this customer. Enter another customer.");
        customer = PromptCustomerByEmail();
        pending = GetPendingOrders(customer);
    }

    Console.WriteLine("\nPending Orders:");
    foreach (Order o in pending)
        Console.WriteLine($"- {o.OrderId}");

    // 3) Pick a valid pending order
    Order orderToModify = null;
    int orderId;

    while (true)
    {
        orderId = PromptInt("Enter Order ID to modify: ");
        orderToModify = FindOrder(customer, orderId);

        if (orderToModify == null)
        {
            Console.WriteLine("Invalid Order ID. Try again.");
            continue;
        }

        if (orderToModify.OrderStatus != "Pending")
        {
            Console.WriteLine("Only Pending orders can be modified. Try again.");
            orderToModify = null;
            continue;
        }

        break;
    }

    // 4) Find restaurant for this order (using your map)
    if (!orderRestaurantMap.ContainsKey(orderId))
    {
        Console.WriteLine("Restaurant mapping not found for this order.");
        Console.WriteLine("Make sure LoadOrders(...) fills orderRestaurantMap.");
        return;
    }

    string restId = orderRestaurantMap[orderId];
    Restaurant restaurant = restaurantById[restId];

    if (restaurant.Menus.Count == 0)
    {
        Console.WriteLine("Restaurant has no menu.");
        return;
    }

    Menu menu = restaurant.Menus[0];

    // 5) Choose what to modify
    string choice;
    while (true)
    {
        Console.WriteLine("\nWhat do you want to modify?");
        Console.WriteLine("1. Ordered Items");
        Console.WriteLine("2. Delivery Address");
        Console.WriteLine("3. Delivery Date/Time");
        Console.Write("Choice: ");
        choice = Console.ReadLine().Trim();

        if (choice == "1" || choice == "2" || choice == "3") break;
        Console.WriteLine("Invalid choice. Enter 1, 2, or 3.");
    }

    if (choice == "1")
    {
        Console.WriteLine("\nReselect items for this order:");
        List<OrderedFoodItem> newItems = SelectOrderedFoodItems(menu);

        // Replace items
        orderToModify.OrderedFoodItem.Clear();
        foreach (var ofi in newItems)
            orderToModify.AddOrderedFoodItem(ofi);

        // Recalculate total
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
        Console.Write(message);
        if (int.TryParse(Console.ReadLine(), out int value))
            return value;
        Console.WriteLine("Invalid number. Try again.");
    }
}

void UpdateOrderRowInCsv(string path, Order o, string custEmail, string restId)
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
        itemParts.Add($"{ofi.ItemName}:{ofi.QtyOrdered}");
    string itemsStr = string.Join(";", itemParts);

    for (int i = 1; i < lines.Length; i++)
    {
        if (string.IsNullOrWhiteSpace(lines[i])) continue;

        string[] parts = lines[i].Split(',');
        if (parts.Length == 0) continue;

        if (int.TryParse(parts[0].Trim(), out int id) && id == o.OrderId)
        {
            // Keep last column (special request) if exists, else blank
            string specialRequest = (parts.Length > 10) ? parts[10].Trim() : "";

            string newLine =
                $"{o.OrderId},{custEmail},{restId}," +
                $"{o.OrderDateTime:dd/MM/yyyy HH:mm}," +
                $"{o.DeliveryDateTime:dd/MM/yyyy HH:mm}," +
                $"{o.DeliveryAddress},{itemsStr}," +
                $"{o.OrderTotal:F2},{o.OrderStatus}," +
                $"{o.OrderPaymentMethod},{specialRequest}";

            lines[i] = newLine;
            File.WriteAllLines(path, lines);
            return;
        }
    }

    Console.WriteLine("Order ID not found in CSV. No update made.");
}
          

           

            

        

        // Basic Feature 8: Gui Ru 

void DeleteOrder()
{
    Console.WriteLine("\nDelete Order");
    Console.WriteLine("============");

    // finding customer detail
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
        Console.WriteLine(order.OrderId);
    }

    Console.Write("Enter Order ID: ");
    if (!int.TryParse(Console.ReadLine(), out int orderID))
    {
        Console.WriteLine("Invalid Order ID format!");
        return;
    }
    // customer name 
    Console.WriteLine();
    Console.WriteLine();


    Order orderToDelete = FindOrder(orderID);
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
        Console.WriteLine($"{itemNum}. {item.ItemName} - {item.ItemPrice}");
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
    }
    else
    {
        Console.WriteLine("Deletion cancelled.");
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
Order FindOrder(int id)
{
    // You need to search the specific customer's orders
    // You can pass the customer as a parameter or find him globally
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
 



