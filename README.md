# prgAssignment



// basic feature 1 : Gui Ru 

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

// basic feature 7 : Anjushree

// basic feature 8 : Gui Ru 
