# prgAssignment

List<Restaurant> RestaurantList = new List<Restaurant>();
List<Customer> CustomerList = new List<Customer>();
Stack<Order> RefundStack = new Stack<Order>();

Console.WriteLine("Welcome to the Gruberoo Food Delivery System");


// Load all data files
LoadRestaurants();
LoadFoodItems();
LoadCustomers();
LoadOrders();

// Main menu loop
while (true)
{
    DisplayMainMenu();
    Console.WriteLine("Enter your choice: ");
    int option = Convert.ToInt32(Console.ReadLine());

    if (option == 1)
    {
        ListAllRestaurantsAndMenuItems();
    }
    else if (option == 2)
    {
        ListAllCustomerAndOrders();
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
        ModifyOrder();
    }
    else if (option == 6)
    {
        DeleteOrder();
    }
    else if (option == 0)
    {
        break;
    }
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

        // ✅ minimal: avoid duplicates + ensure dictionary exists
        restaurants.Clear();
        restaurantById.Clear();

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

                // ✅ minimal: each restaurant has at least one menu (required + used by Feature 5)
                restaurant.AddMenu(new Menu("M001", "Main Menu"));

                restaurants.Add(restaurant);

                // ✅ minimal: needed by Feature 2/7
                restaurantById[restaurantID] = restaurant;

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
        if (!File.Exists("fooditems - Copy.csv"))
        {
            Console.WriteLine("fooditems - Copy.csv file not found!");
            return;
        }

        
        string[] lines = File.ReadAllLines("fooditems - Copy.csv");
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

                //use dictionary built in LoadRestaurants()
                if (restaurantById.ContainsKey(restaurantID))
                {
                    Restaurant restaurant = restaurantById[restaurantID];

                    //FoodItem constructor needs 4 params
                    FoodItem foodItem = new FoodItem(itemName, description, price, "");

                    //add to the restaurant's first menu 
                    restaurant.Menus[0].AddFoodItem(foodItem);

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
