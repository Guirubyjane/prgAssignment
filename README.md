using System;
using System.Collections.Generic;

namespace S10272786F_PRG2Assignment
{
    internal class Menu
    {
        private string menuId;
        private string menuName;
        public List<FoodItem> foodItems { get; set; } = new List<FoodItem>();

        public Menu(string menuId, string menuName)
        {
            this.menuId = menuId;
            this.menuName = menuName;
        }

        public void AddFoodItem(FoodItem foodItem)
        {
            foodItems.Add(foodItem);
        }

        public bool RemoveFoodItem(FoodItem foodItem)
        {
            return foodItems.Remove(foodItem);
        }

        public void DisplayFoodItems()
        {
            for (int i = 0; i < foodItems.Count; i++)
            {
                Console.WriteLine($"{i + 1}. {foodItems[i]}");
            }
        }

        public override string ToString()
        {
            return $"Menu: {menuName} ({menuId})";
        }

      
    }
}

