# prgAssignment
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Text;
using System.Threading.Tasks;
using System.Xml.Linq;

//==========================================================
// Student Number : S10274330A 
// Student Name : Gui Ru
// Partner Name : Anjushree
//==========================================================

namespace S10274330_PRG2Assignment
{
    class Customer
    {
        // attribute
        private string emailAddress;
        private string customerName;

        // properties
        public string EmailAddress { get; set; }
        public string CustomerName { get; set; }

        // 1 customer --> 1..* orders 
        public List<Order> OrderList { get; set; } = new List<Order>();

        // constructors
        public Customer() { }
        public Customer(string ea, string cn)
        {
            EmailAddress = ea;
            CustomerName = cn;
        }

        // method
        public void AddOrder(Order order)
        {
            if (order != null)
            {
                OrderList.Add(order);
            }
        }

        public void DisplayAllOrders()
        {
            if (OrderList.Count == 0)
            {
                Console.WriteLine($"No orders found for {CustomerName}");
                return;
            }

            Console.WriteLine($"\nOrders for {CustomerName}");
            Console.WriteLine($"Email Address: {EmailAddress}");
            Console.WriteLine("==========================================================");
            foreach (Order order in OrderList)
            {
                Console.WriteLine(order.ToString());
            }
        }

        public bool RemoveOrder(Order order)
        {
            if (OrderList.Contains(order))
            {
                OrderList.Remove(order);
                return true;
            }
            return false;
        }

        public override string ToString()
        {
            return $"{CustomerName} ({EmailAddress}) - {OrderList.Count} orders";
        }

    }
}
