using System.Net.Http.Headers;

namespace S10274330_PRG2Assignment
{
    internal class Restaurant
    {
        private string restaurantId;
        private string restaurantName;
        private string restaurantEmail;

        public string RestaurantId
        { get { return restaurantId; } set { restaurantId = value; } }
        public string RestaurantName
        { get { return restaurantName; } set { restaurantName = value; } }
        public string RestaurantEmail
        {
            get { return restaurantEmail; }
            set { restaurantEmail = value; }
        }

        public List<Menu> Menus { get; set; } = new List<Menu>();
        public Queue<Order> Orders { get; set; } = new Queue<Order>();
        public List<SpecialOffer> SpecialOffers { get; set; } = new List<SpecialOffer>();

        public Restaurant(string restaurantId, string restaurantName, string restaurantEmail)
        {
            RestaurantId = restaurantId;
            RestaurantName = restaurantName;
            RestaurantEmail = restaurantEmail;
        }

        public void AddMenu(Menu menu)
        {
            Menus.Add(menu);
        }
        public bool RemoveMenu(Menu menu)
        {
            return Menus.Remove(menu);
        }
        public void DisplayMenu()
        {
            foreach (Menu menu in Menus)
            {
                Console.WriteLine(menu);
                menu.DisplayFoodItems();
            }
        }
        public void DisplayOrders()
        {
            foreach (Order order in Orders)
            {
                Console.WriteLine(order);
            }
        }
        public void DisplaySpecialOffers()
        {
            foreach (SpecialOffer offer in SpecialOffers)
            {
                Console.WriteLine(offer);
            }
        }
        public override string ToString()
        {
            return $"{RestaurantName} ({RestaurantId}) - {RestaurantEmail}";
        }
    }
}
