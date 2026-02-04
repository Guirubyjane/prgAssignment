# prgAssignment
class Order
{
    private int orderId;
    private DateTime orderDateTime;
    private double orderTotal;
    private string orderStatus;
    private DateTime deliveryDateTime;
    private string deliveryAddress;
    private string orderPaymentMethod;
    private bool orderPaid;

    public int OrderId { get; set; }
    public DateTime OrderDateTime { get; set; }
    public double OrderTotal { get; set; }
    public string OrderStatus { get; set; }
    public DateTime DeliveryDateTime { get; set; }
    public string DeliveryAddress { get; set; }
    public string OrderPaymentMethod { get; set; }
    public bool OrderPaid { get; set; }

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
    public void AddItem(FoodItem item, int quantity)
    {
        if (OrderedItems.ContainsKey(item))
        {
            OrderedItems[item] += quantity;
        }
        else
        {
            OrderedItems.Add(item, quantity);
            FoodItems.Add(item);
        }
    }

    public void RemoveItem(FoodItem item)
    {
        if (OrderedItems.ContainsKey(item))
        {
            OrderedItems.Remove(item);
            FoodItems.Remove(item);
        }
    }

    public void UpdateItem(FoodItem item, int newQuantity)
    {
        if (OrderedItems.ContainsKey(item))
        {
            OrderedItems[item] = newQuantity;
        }
    }

    public double CalculateTotal()
    {
        double subtotal = 0;
        foreach (var item in OrderedItems)
        {
            subtotal += item.Key.Price * item.Value;
        }

        // Add delivery fee
        const double DELIVERY_FEE = 5.00;
        TotalAmount = subtotal + DELIVERY_FEE;
        return TotalAmount;
    }

    public void UpdateStatus(string newStatus)
    {
        Status = newStatus;
    }

    public void ModifyDeliveryDateTime(DateTime newDateTime)
    {
        DeliveryDateTime = newDateTime;
    }

    public void ModifyDeliveryAddress(string newAddress)
    {
        DeliveryAddress = newAddress;
    }

    public void SetSpecialRequest(string request)
    {
        SpecialRequest = request;
    }

    public override string ToString()
    {
        return $"Order {OrderID}: Customer: {Customer?.Name}, Restaurant: {Restaurant?.Name}, " +
               $"Delivery: {DeliveryDateTime:dd/MM/yyyy HH:mm}, Amount: ${TotalAmount:F2}, Status: {Status}";
    }

    // Display detailed order information
    public void DisplayOrderDetails()
    {
        Console.WriteLine($"Order ID: {OrderID}");
        Console.WriteLine($"Customer: {Customer?.Name}");
        Console.WriteLine("Ordered Items:");
        int itemNum = 1;
        foreach (var item in OrderedItems)
        {
            Console.WriteLine($"{itemNum}. {item.Key.ItemName} - {item.Value}");
            itemNum++;
        }
        Console.WriteLine($"Delivery date/time: {DeliveryDateTime:dd/MM/yyyy HH:mm}");
        Console.WriteLine($"Delivery Address: {DeliveryAddress}");
        if (!string.IsNullOrEmpty(SpecialRequest))
        {
            Console.WriteLine($"Special Request: {SpecialRequest}");
        }
        Console.WriteLine($"Total Amount: ${TotalAmount:F2}");
        Console.WriteLine($"Order Status: {Status}");
    }

    // Convert order to CSV format for saving
    public string ToCSV()
    {
        string itemsStr = "";
        foreach (var item in OrderedItems)
        {
            itemsStr += $"{item.Key.ItemName}:{item.Value};";
        }
        itemsStr = itemsStr.TrimEnd(';');

        return $"{OrderID},{Customer.Email},{Restaurant.RestaurantID}," +
               $"{OrderDateTime:dd/MM/yyyy HH:mm},{DeliveryDateTime:dd/MM/yyyy HH:mm}," +
               $"{DeliveryAddress},{itemsStr},{TotalAmount:F2},{Status},{PaymentMethod},{SpecialRequest}";
    }
}
