# prgAssignment



// basic feature 1 : Gui Ru 

// basic feature 2 : Anjushree

// basic feature 3 : Anjushree

static List<Restaurant> restaurants = new List<Restaurant>();
static Dictionary<string, Restaurant> restaurantById =
    new Dictionary<string, Restaurant>();

static void Main(string[] args)
{
    LoadRestaurants("restaurants.csv");
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


// basic feature 4 : Gui Ru

// basic feature 5 : Anjushree

// basic feature 6 : Gui Ru 

// basic feature 7 : Anjushree

// basic feature 8 : Gui Ru 
