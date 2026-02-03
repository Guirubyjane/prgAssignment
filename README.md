# prgAssignment


namespace S10272786F_PRG2Assignment
{
    internal class OrderedFoodItem : FoodItem
    {
        private int qytOrdered;
        private double subTotal;

        public int QtyOrdered
            { get { return qytOrdered; } set { qytOrdered = value; } }
        public double SubTotal
            { get { return subTotal; } set { subTotal = value; } }

        public OrderedFoodItem(string itemName, string itemDesc, double itemPrice, string customise, int qtyOrdered) : base(itemName, itemDesc, itemPrice, customise)
        {
            qtyOrdered = QtyOrdered;
            subTotal = CalculateSubTotal();
        }
        public double CalculateSubTotal()
        {
            return ItemPrice * QtyOrdered;
        }

        public override string ToString()
        {
            return $"{ItemName} - {QtyOrdered} x ${ItemPrice:F2} = ${CalculateSubTotal():F2}";
        }
    }
}
