# prgAssignment

 class Customer
 {
     private string emailAddress;
     private string customerName;
     public string EmailAddress { get; set; }
     public string CustomerName { get; set; }

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
             order.Customer = this;
         }
     }

     public void RemoveOrder(Order order)
     {
         if (OrderList.Contains(order))
         {
             OrderList.Remove(order);
         }
     }

     public Order FindOrder(int orderID)
     {
         foreach (Order order in OrderList)
         {
             if (order.OrderID == orderID)
             {
                 return order;
             }
         }
         return null;
     }

     public List<Order> GetPendingOrders()
     {
         List<Order> pendingOrders = new List<Order>();
         foreach (Order order in OrderList)
         {
             if (order.Status == "Pending")
             {
                 pendingOrders.Add(order);
             }
         }
         return pendingOrders;
     }

     public List<Order> GetOrdersByStatus(string status)
     {
         return OrderList.Where(o => o.Status == status).ToList();
     }

     public void DisplayAllOrders()
     {
         if (OrderList.Count == 0)
         {
             Console.WriteLine($"No orders found for {Name}");
             return;
         }

         Console.WriteLine($"\nOrders for {Name} ({Email}):");
         Console.WriteLine("=".PadRight(80, '='));
         foreach (Order order in OrderList)
         {
             Console.WriteLine(order.ToString());
         }
     }

     public void DisplayPendingOrders()
     {
         List<Order> pendingOrders = GetPendingOrders();

         if (pendingOrders.Count == 0)
         {
             Console.WriteLine($"No pending orders for {Name}");
             return;
         }

         Console.WriteLine($"Pending Orders for {Name}:");
         foreach (Order order in pendingOrders)
         {
             Console.WriteLine(order.OrderID);
         }
     }

     public double GetTotalSpent()
     {
         double total = 0;
         foreach (Order order in OrderList)
         {
             if (order.Status == "Delivered")
             {
                 total += order.TotalAmount;
             }
         }
         return total;
     }

     public int GetOrderCount()
     {
         return OrderList.Count;
     }

     public int GetDeliveredOrderCount()
     {
         return OrderList.Count(o => o.Status == "Delivered");
     }

     public override string ToString()
     {
         return $"{Name} ({Email}) - {OrderList.Count} orders";
     }

     // Convert to CSV format
     public string ToCSV()
     {
         return $"{Name},{Email}";
     }
 }
