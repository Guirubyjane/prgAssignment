# prgAssignment

class SpecialOffer
{
    private string offerCode;
    private string offerDesc;
    private double discount;

    public string OfferCode { get; set; }
    public string OfferDesc { get; set; }
    public double Discount { get; set; }

    public SpecialOffer() { }
    public SpecialOffer(string oc, string od, double d ) 
    {
        offerCode = oc;
        offerDesc = od;
        discount = d;
    }

    // method 
    public double ApplyDiscount(double originalAmount)
    {
        if (DiscountAmount > 0)
        {
            double discountValue = originalAmount * (DiscountAmount / 100);
            return originalAmount - discountValue;
        }
        return originalAmount;
    }

    public bool IsValidOffer()
    {
        return !string.IsNullOrEmpty(OfferCode) && !string.IsNullOrEmpty(Description);
    }

    public override string ToString()
    {
        if (DiscountAmount > 0)
        {
            return $"{OfferCode}: {Description} - {DiscountAmount}% off";
        }
        else
        {
            return $"{OfferCode}: {Description}";
        }
    }

    // Display offer details
    public void DisplayOfferDetails()
    {
        Console.WriteLine($"Offer Code: {OfferCode}");
        Console.WriteLine($"Description: {Description}");
        if (DiscountAmount > 0)
        {
            Console.WriteLine($"Discount: {DiscountAmount}%");
        }
        else
        {
            Console.WriteLine("Discount: N/A (Free delivery or other benefits)");
        }
    }

    // Convert to CSV format
    public string ToCSV()
    {
        string restaurantID = Restaurant != null ? Restaurant.RestaurantID : "";
        return $"{restaurantID},{OfferCode},{Description},{DiscountAmount}";
    }

    public string ToString()
    {
        return "Offer Code:" + OfferCode + " Offer Desc: " + OfferDesc + " Discount: " + Discount;
    }

}
