

namespace S10272786F_PRG2Assignment
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

        private List<Menu> menus;
        private Queue<Order> orders;
        private List<SpecialOffer> specialOffers;

        public Restaurant(string restaurantId, string restaurantName, string restaurantEmail)
        {
           restaurantId = RestaurantId;
           restaurantName = RestaurantName;
           restaurantEmail = RestaurantEmail;

            menus = new List<Menu>();
            orders = new Queue<Order>();
            specialOffers = new List<SpecialOffer>();
        }

        public void AddMenu(Menu menu)
        {
            menus.Add(menu);
        }
        public bool RemoveMenu(Menu menu)
        {
            return menus.Remove(menu);
        }
        public void DisplayMenu()
        {
            foreach (Menu menu in menus)
            {
                Console.WriteLine(menu);
                menu.DisplayFoodItems();
            }
        }
        public void DisplayOrders()
        {
            foreach (Order order in orders)
            {
                Console.WriteLine(order);
            }
        }
        public void DisplaySpecialOffers()
        {
            foreach (SpecialOffer offer in specialOffers)
            {
                Console.WriteLine(offer);
            }
        }
        public override string ToString()
        {
            return $"{restaurantName} ({restaurantId}) - {restaurantEmail}";
        }
    }
}
