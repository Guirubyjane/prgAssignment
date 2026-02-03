# prgAssignment




namespace S10272786F_PRG2Assignment
{
    internal class FoodItem
    {
        private string itemName;
        private string itemDesc;
        private double itemPrice;
        private string customise;

        public string ItemName
        { get { return itemName; } set { itemName = value; } }
        public string ItemDesc
        { get { return itemDesc; } set { itemDesc = value; } }
        public double ItemPrice
        { get { return itemPrice; } set { itemPrice = value; } }
        public string Customise
        { get { return customise; } set { customise = value; } }

        public FoodItem(string itemName, string itemDesc, double itemPrice, string customise)
        {
            itemName = ItemName;
            itemDesc = ItemDesc;
            itemPrice = ItemPrice;
            customise = Customise;
        }

        public override string ToString()
        {
            return $"{itemName}: {itemDesc} - ${itemPrice:F2}";
        }

    }
}
