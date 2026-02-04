# prgAssignment



// basic feature 1 : Gui Ru 
// ============================================
// BASIC FEATURE 1: Load Restaurants and Food Items
// ============================================
static void LoadRestaurants()
{
    try
    {
        if (!File.Exists("restaurants.csv"))
        {
            Console.WriteLine("restaurants.csv file not found!");
            return;
        }

        string[] lines = File.ReadAllLines("restaurants.csv");
        int count = 0;

        // Skip header line
        for (int i = 1; i < lines.Length; i++)
        {
            string[] parts = lines[i].Split(',');
            if (parts.Length >= 3)
            {
                string restaurantID = parts[0].Trim();
                string name = parts[1].Trim();
                string email = parts[2].Trim();

                Restaurant restaurant = new Restaurant(restaurantID, name, email);
                restaurants.Add(restaurant);
                count++;
            }
        }

        Console.WriteLine($"{count} restaurants loaded!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading restaurants: {ex.Message}");
    }
}

static void LoadFoodItems()
{
    try
    {
        if (!File.Exists("fooditems.csv"))
        {
            Console.WriteLine("fooditems.csv file not found!");
            return;
        }

        string[] lines = File.ReadAllLines("fooditems.csv");
        int count = 0;

        // Skip header line
        for (int i = 1; i < lines.Length; i++)
        {
            string[] parts = lines[i].Split(',');
            if (parts.Length >= 4)
            {
                string restaurantID = parts[0].Trim();
                string itemName = parts[1].Trim();
                string description = parts[2].Trim();
                double price = double.Parse(parts[3].Trim());

                // Find the restaurant
                Restaurant restaurant = FindRestaurant(restaurantID);
                if (restaurant != null)
                {
                    FoodItem foodItem = new FoodItem(itemName, description, price);
                    restaurant.AddFoodItem(foodItem);
                    count++;
                }
            }
        }

        Console.WriteLine($"{count} food items loaded!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading food items: {ex.Message}");
    }
}

static void LoadCustomers()
{
    try
    {
        if (!File.Exists("customers.csv"))
        {
            Console.WriteLine("customers.csv file not found!");
            return;
        }

        string[] lines = File.ReadAllLines("customers.csv");
        int count = 0;

        // Skip header line
        for (int i = 1; i < lines.Length; i++)
        {
            string[] parts = lines[i].Split(',');
            if (parts.Length >= 2)
            {
                string name = parts[0].Trim();
                string email = parts[1].Trim();

                Customer customer = new Customer(name, email);
                customers.Add(customer);
                count++;
            }
        }

        Console.WriteLine($"{count} customers loaded!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading customers: {ex.Message}");
    }
}

static void LoadOrders()
{
    try
    {
        if (!File.Exists("orders.csv"))
        {
            Console.WriteLine("orders.csv file not found!");
            return;
        }

        string[] lines = File.ReadAllLines("orders.csv");
        int count = 0;

        // Skip header line
        for (int i = 1; i < lines.Length; i++)
        {
            string line = lines[i];
            if (string.IsNullOrWhiteSpace(line)) continue;

            string[] parts = line.Split(',');
            if (parts.Length >= 9)
            {
                int orderID = int.Parse(parts[0].Trim());
                string customerEmail = parts[1].Trim();
                string restaurantID = parts[2].Trim();
                DateTime orderDateTime = DateTime.ParseExact(parts[3].Trim(), "dd/MM/yyyy HH:mm", CultureInfo.InvariantCulture);
                DateTime deliveryDateTime = DateTime.ParseExact(parts[4].Trim(), "dd/MM/yyyy HH:mm", CultureInfo.InvariantCulture);
                string deliveryAddress = parts[5].Trim();
                string itemsStr = parts[6].Trim();
                double totalAmount = double.Parse(parts[7].Trim());
                string status = parts[8].Trim();
                string paymentMethod = parts.Length > 9 ? parts[9].Trim() : "CC";
                string specialRequest = parts.Length > 10 ? parts[10].Trim() : "";

                // Find customer and restaurant
                Customer customer = FindCustomer(customerEmail);
                Restaurant restaurant = FindRestaurant(restaurantID);

                if (customer != null && restaurant != null)
                {
                    Order order = new Order(orderID, orderDateTime, deliveryDateTime,
                                           deliveryAddress, status, totalAmount,
                                           paymentMethod, customer, restaurant);

                    order.SpecialRequest = specialRequest;

                    // Parse and add food items
                    if (!string.IsNullOrEmpty(itemsStr))
                    {
                        string[] items = itemsStr.Split(';');
                        foreach (string item in items)
                        {
                            string[] itemParts = item.Split(':');
                            if (itemParts.Length == 2)
                            {
                                string itemName = itemParts[0].Trim();
                                int quantity = int.Parse(itemParts[1].Trim());

                                FoodItem foodItem = restaurant.FindFoodItem(itemName);
                                if (foodItem != null)
                                {
                                    order.AddItem(foodItem, quantity);
                                }
                            }
                        }
                    }

                    // Add order to customer and restaurant
                    customer.AddOrder(order);
                    restaurant.AddOrder(order);
                    count++;

                    // Update next order ID
                    if (orderID >= nextOrderID)
                    {
                        nextOrderID = orderID + 1;
                    }
                }
            }
        }

        Console.WriteLine($"{count} orders loaded!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error loading orders: {ex.Message}");
    }
}

// basic feature 2 : Anjushree

// basic feature 3 : Anjushree

 static void Main(string[] args)
 {
     LoadRestaurants("restaurants.csv");
     LoadFoodItems("fooditems - Copy.csv");
     ListAllRestaurantsAndMenuItems();
 }

 static void LoadRestaurants(string path)
 {
     foreach (string line in File.ReadAllLines(path))
     {
         if (string.IsNullOrWhiteSpace(line)) continue;
         if (line.StartsWith("RestaurantId")) continue;

string[] parts = line.Split(',');
 string id = parts[0].Trim();
         string name = parts[1].Trim();
         string email = parts[2].Trim();

Restaurant r = new Restaurant(id, name, email);
 r.AddMenu(new Menu("M001", "Main Menu"));

restaurants.Add(r);
restaurantById[id] = r;
     }
     Console.WriteLine($"{restaurants.Count} restaurants loaded!");
 }
 static void LoadFoodItems(string path)
 {
     foreach (string line in File.ReadAllLines(path))
     {
         if (string.IsNullOrWhiteSpace(line)) continue;
         if (line.StartsWith("RestaurantId")) continue;

tring[] parts = line.Split(',');
if (parts.Length < 4) continue;

string restaurantId = parts[0].Trim();
string itemName = parts[1].Trim();
 string desc = parts[2].Trim();
double price = double.Parse(parts[3].Trim());

if (!restaurantById.ContainsKey(restaurantId)) continue;

 FoodItem foodItem = new FoodItem(itemName, desc, price, "");

 // Add food item to the restaurant's main menu
 restaurantById[restaurantId].Menus[0].AddFoodItem(foodItem);
     }
 }

 static void ListAllRestaurantsAndMenuItems()
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


// basic feature 4 : Gui Ru

// basic feature 5 : Anjushree

// basic feature 6 : Gui Ru 
// ============================================
// BASIC FEATURE 6: Process an order
// ============================================
static void ProcessOrder()
{
    Console.WriteLine("\nProcess Order");
    Console.WriteLine("=============");

    Console.Write("Enter Restaurant ID: ");
    string restaurantID = Console.ReadLine().Trim();

    Restaurant restaurant = FindRestaurant(restaurantID);
    if (restaurant == null)
    {
        Console.WriteLine("Restaurant not found!");
        return;
    }

    if (restaurant.OrderQueue.Count == 0)
    {
        Console.WriteLine("No orders in the queue for this restaurant.");
        return;
    }

    // Process orders in the queue
    Queue<Order> tempQueue = new Queue<Order>();
    bool hasProcessed = false;

    while (restaurant.OrderQueue.Count > 0)
    {
        Order order = restaurant.OrderQueue.Dequeue();

        Console.WriteLine($"\nOrder {order.OrderID}:");
        Console.WriteLine($"Customer: {order.Customer.Name}");
        Console.WriteLine("Ordered Items:");
        int itemNum = 1;
        foreach (var item in order.OrderedItems)
        {
            Console.WriteLine($"{itemNum}. {item.Key.ItemName} - {item.Value}");
            itemNum++;
        }
        Console.WriteLine($"Delivery date/time: {order.DeliveryDateTime:dd/MM/yyyy HH:mm}");
        Console.WriteLine($"Total Amount: ${order.TotalAmount:F2}");
        Console.WriteLine($"Order Status: {order.Status}");

        Console.Write("\n[C]onfirm / [R]eject / [S]kip / [D]eliver: ");
        string action = Console.ReadLine().Trim().ToUpper();

        switch (action)
        {
            case "C":
                if (order.Status == "Pending")
                {
                    order.UpdateStatus("Preparing");
                    Console.WriteLine($"Order {order.OrderID} confirmed. Status: Preparing");
                    hasProcessed = true;
                }
                else
                {
                    Console.WriteLine($"Cannot confirm order. Current status: {order.Status}");
                }
                tempQueue.Enqueue(order);
                break;

            case "R":
                if (order.Status == "Pending")
                {
                    order.UpdateStatus("Rejected");
                    refundStack.Push(order);
                    Console.WriteLine($"Order {order.OrderID} rejected. Refund of ${order.TotalAmount:F2} processed.");
                    hasProcessed = true;
                }
                else
                {
                    Console.WriteLine($"Cannot reject order. Current status: {order.Status}");
                    tempQueue.Enqueue(order);
                }
                break;

            case "S":
                if (order.Status == "Cancelled")
                {
                    Console.WriteLine($"Order {order.OrderID} skipped (Cancelled).");
                }
                else
                {
                    Console.WriteLine($"Order {order.OrderID} skipped.");
                }
                tempQueue.Enqueue(order);
                break;

            case "D":
                if (order.Status == "Preparing")
                {
                    order.UpdateStatus("Delivered");
                    Console.WriteLine($"Order {order.OrderID} delivered. Status: Delivered");
                    hasProcessed = true;
                }
                else
                {
                    Console.WriteLine($"Cannot deliver order. Current status: {order.Status}");
                }
                tempQueue.Enqueue(order);
                break;

            default:
                Console.WriteLine("Invalid action. Order skipped.");
                tempQueue.Enqueue(order);
                break;
        }
    }

    // Restore the queue
    while (tempQueue.Count > 0)
    {
        restaurant.OrderQueue.Enqueue(tempQueue.Dequeue());
    }

    if (!hasProcessed)
    {
        Console.WriteLine("\nNo orders were processed.");
    }
}

// basic feature 7 : Anjushree

// basic feature 8 : Gui Ru 
// ============================================
// BASIC FEATURE 8: Delete an existing order
// ============================================
static void DeleteOrder()
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

    List<Order> pendingOrders = customer.GetPendingOrders();
    if (pendingOrders.Count == 0)
    {
        Console.WriteLine("No pending orders for this customer.");
        return;
    }

    Console.WriteLine("Pending Orders:");
    foreach (Order order in pendingOrders)
    {
        Console.WriteLine(order.OrderID);
    }

    Console.Write("Enter Order ID: ");
    if (!int.TryParse(Console.ReadLine(), out int orderID))
    {
        Console.WriteLine("Invalid Order ID format!");
        return;
    }

    Order orderToDelete = customer.FindOrder(orderID);
    if (orderToDelete == null)
    {
        Console.WriteLine("Order not found!");
        return;
    }

    if (orderToDelete.Status != "Pending")
    {
        Console.WriteLine($"Cannot delete order. Current status: {orderToDelete.Status}");
        Console.WriteLine("Only pending orders can be deleted.");
        return;
    }

    // Display order details
    Console.WriteLine($"\nCustomer: {customer.Name}");
    Console.WriteLine("Ordered Items:");
    int itemNum = 1;
    foreach (var item in orderToDelete.OrderedItems)
    {
        Console.WriteLine($"{itemNum}. {item.Key.ItemName} - {item.Value}");
        itemNum++;
    }
    Console.WriteLine($"Delivery date/time: {orderToDelete.DeliveryDateTime:dd/MM/yyyy HH:mm}");
    Console.WriteLine($"Total Amount: ${orderToDelete.TotalAmount:F2}");
    Console.WriteLine($"Order Status: {orderToDelete.Status}");

    Console.Write("\nConfirm deletion? [Y/N]: ");
    string confirm = Console.ReadLine().Trim().ToUpper();

    if (confirm == "Y")
    {
        orderToDelete.UpdateStatus("Cancelled");
        refundStack.Push(orderToDelete);
        Console.WriteLine($"Order {orderID} cancelled. Refund of ${orderToDelete.TotalAmount:F2} processed.");
    }
    else
    {
        Console.WriteLine("Deletion cancelled.");
    }
}
