# prgAssignment



// basic feature 1 

// basic feature 2 

// basic feature 3 

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


// basic feature 4

// basic feature 5

// basic feature 6

// basic feature 7

// basic feature 8 
