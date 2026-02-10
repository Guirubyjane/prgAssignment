# prgAssignment
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

//==========================================================
// Student Number : S10274330A
// Student Name : Gui Ru
// Partner Name : Anjushree
//==========================================================

namespace S10274330_PRG2Assignment
{
    class SpecialOffer
    {
        private string offerCode;
        private string offerDesc;
        private double discount;

        public string OfferCode { get; set; }
        public string OfferDesc { get; set; }
        public double Discount { get; set; }

        // constructor
        public SpecialOffer() { }
        public SpecialOffer(string oc, string od, double d ) 
        {
            offerCode = oc;
            offerDesc = od;
            discount = d;
        }

        // method 
     
        public override string ToString()
        {
            if (Discount > 0)
            {
                return $"{OfferCode}: {OfferDesc} - {Discount}% off";
            }
            else
            {
                return $"{OfferCode}: {OfferDesc}";
            }
        }
    }
}

