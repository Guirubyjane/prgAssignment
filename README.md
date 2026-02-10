# prgAssignment
using S10274330_PRG2Assignment;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.AccessControl;
using System.Text;
using System.Threading.Tasks;

//==========================================================
// Student Number : S10274330A
// Student Name : Gui Ru
// Partner Name : Anjushree
//==========================================================

namespace S10274330_PRG2Assignment
{
    class Order
    {
        // attributes
        private int orderId;
        private DateTime orderDateTime;
        private double orderTotal;
        private string orderStatus;
        private DateTime deliveryDateTime;
        private string deliveryAddress;
        private string orderPaymentMethod;
        private bool orderPaid;

        // properties
        public int OrderId { get; set; }
        public DateTime OrderDateTime { get; set; }
        public double OrderTotal { get; set; }
        public string OrderStatus { get; set; }
        public DateTime DeliveryDateTime { get; set; }
        public string DeliveryAddress { get; set; }
        public string OrderPaymentMethod { get; set; }
        public bool OrderPaid { get; set; }

        // 1 order --> 1..* OrderedFoodItem
        public List<OrderedFoodItem> OrderedFoodItem { get; set; } = new List<OrderedFoodItem>();

        // 1..* order <--> 1 Customer
        public Customer Customer { get; set; }

        // 0..* Order <--> 1 Restaurant
        public Restaurant Restaurant { get; set; }

        //0..*  Order --> 1 SpecialOffer           
        public SpecialOffer SpecialOffer { get; set; }

        // constructors
        public Order() { }
        public Order(int oi, DateTime odt, double ot, string os, DateTime ddt, string da, string opm, bool op)
        {
            OrderId = oi;
            OrderDateTime = odt;
            OrderTotal = ot;
            OrderStatus = os;
            DeliveryDateTime = ddt;
            DeliveryAddress = da;
            OrderPaymentMethod = opm;
            OrderPaid = op;
        }

        // method 

        public double CalculateOrderTotal()
        {
            double total = 0;
            foreach (OrderedFoodItem ofi in OrderedFoodItem)
            {
                total += ofi.CalculateSubTotal();
            }

            total += 5;                     // delivery fee

            if (SpecialOffer != null)
            {
                double discountValue = total * (SpecialOffer.Discount / 100);
                 total -= discountValue;
            }

            return total;
        }

        public void AddOrderedFoodItem(OrderedFoodItem ofi)
        {
            if (ofi != null)
            {
                OrderedFoodItem.Add(ofi);
            }
        }

        public bool RemoveOrderedFoodItem(OrderedFoodItem ofi)
        {
            if (ofi != null && OrderedFoodItem.Contains(ofi))
            {
                OrderedFoodItem.Remove(ofi);
                return true;
            }
            return false;
        }

        public void DisplayOrderedFoodItem()
        {
            Console.WriteLine("\nOrdered Food Items:");
            Console.WriteLine("===================");

            if (OrderedFoodItem.Count == 0)
            {
                Console.WriteLine("No items in this order.");
                return;
            }

            int itemNumber = 1;
            foreach (OrderedFoodItem ofi in OrderedFoodItem)
            {
                double itemSubtotal = ofi.CalculateSubTotal();
                Console.WriteLine($"{itemNumber}. {ofi.ToString()}");
                itemNumber++;
            }

            Console.WriteLine($"\nDelivery Fee: $5.00");

            if (SpecialOffer != null && SpecialOffer.Discount > 0)
            {
                Console.WriteLine($"Discount ({SpecialOffer.OfferCode}): -{SpecialOffer.Discount}%");
            }

            OrderTotal = CalculateOrderTotal();

            Console.WriteLine($"Order Total: ${OrderTotal:F2}");
        }

        public override string ToString()
        {
            // check if customer and customer name is null, then put as unknown customer
            string customerName = Customer?.CustomerName ?? "Unknown Customer";
            string restaurantName = Restaurant?.RestaurantName ?? "Unknown Restaurant";

            //ID, Customer, Restaurant, Time, Amount, Status
            return $"{OrderId,-9} " +
                   $"{customerName,-15} " +
                   $"{restaurantName,-15} " +
                   $"{DeliveryDateTime,-20:dd/MM/yyyy HH:mm} " +
                   $"${OrderTotal,-8:F2} " +
                   $"{OrderStatus}";
        }
    }

}


