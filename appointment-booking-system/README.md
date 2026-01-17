# 📅 Appointment Booking System

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-008CDD?style=for-the-badge&logo=stripe&logoColor=white)

> **Multi-tenant scheduling platform with smart slot management, payment integration, and automated notifications**

## 📋 Problem Statement

Service-based businesses face scheduling challenges:

- 📅 **Double Bookings**: Manual scheduling leads to conflicts
- 💸 **No-Shows**: Lost revenue from missed appointments
- 📞 **Manual Reminders**: Time-consuming customer notifications
- 💳 **Payment Issues**: Collecting deposits is difficult
- 🏢 **Multi-Location**: Managing multiple branches is complex

## 💡 Solution: Smart Appointment Booking System

A comprehensive multi-tenant platform that automates scheduling, payments, and notifications for service businesses like doctors, salons, and consultants.

### Key Features

- 🎯 **Smart Slot Manager**: Intelligent algorithm prevents double bookings
- 💳 **Stripe Integration**: Secure payment processing and deposits
- 📲 **SMS Notifications**: Automated reminders via AWS SNS
- 🏢 **Multi-Tenant**: Support for multiple businesses/locations
- 📊 **Analytics Dashboard**: Booking insights and revenue tracking
- 🔄 **Recurring Appointments**: Support for weekly/monthly bookings
- ⏰ **Buffer Time**: Automatic gaps between appointments

## 🏗️ Architecture

\`\`\`mermaid
graph TB
    A[React Frontend] -->|API Calls| B[Express Backend]
    B -->|CRUD| C[MongoDB]
    B -->|Send SMS| D[AWS SNS]
    B -->|Process Payment| E[Stripe API]
    F[Slot Manager] -->|Check Availability| C
    B -->|Use| F
    G[Cron Jobs] -->|Trigger| H[Reminder Service]
    H -->|Send| D

    style A fill:#61DAFB
    style B fill:#000000
    style C fill:#47A248
    style E fill:#008CDD
\`\`\`

### Booking Flow

1. **Customer selects service** and preferred date/time
2. **Slot Manager checks availability** in real-time
3. **Customer provides details** and makes payment via Stripe
4. **Booking confirmed** and stored in MongoDB
5. **SMS confirmation** sent immediately via AWS SNS
6. **Reminder sent** 24 hours before appointment
7. **Post-appointment** feedback request sent

## 📁 Project Structure

\`\`\`
appointment-booking-system/
├── backend/
│   ├── controllers/
│   │   ├── appointmentController.js   # Booking logic
│   │   ├── userController.js          # User management
│   │   └── paymentController.js       # Stripe integration
│   ├── models/
│   │   ├── Appointment.js             # Appointment schema
│   │   ├── User.js                    # User schema
│   │   └── Business.js                # Business schema
│   ├── services/
│   │   ├── slot_manager.js            # Smart slot algorithm
│   │   ├── sms_service.js             # AWS SNS integration
│   │   └── payment_service.js         # Stripe wrapper
│   ├── routes/
│   │   └── api.js                     # API routes
│   └── server.js                      # Express server
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Calendar.jsx           # Booking calendar
│   │   │   ├── ServiceList.jsx        # Available services
│   │   │   ├── BookingForm.jsx        # Appointment form
│   │   │   └── Dashboard.jsx          # Admin dashboard
│   │   ├── pages/
│   │   │   ├── Home.jsx
│   │   │   ├── Booking.jsx
│   │   │   └── Admin.jsx
│   │   └── App.jsx
│   └── package.json
└── README.md
\`\`\`

## 🚀 Setup & Deployment

### Prerequisites

- Node.js 16+
- MongoDB (local or Atlas)
- Stripe account
- AWS account (for SNS)
- npm or yarn

### Backend Setup

1. **Clone and install**
   \`\`\`bash
   git clone <https://github.com/Abhinandansinha01/portfolio-2026.git>
   cd appointment-booking-system/backend
   npm install
   \`\`\`

2. **Configure environment variables**
   \`\`\`bash

   # Create .env file

   cat > .env << EOF
   PORT=5000
   MONGODB_URI=mongodb://localhost:27017/appointments
   STRIPE_SECRET_KEY=sk_test_your_stripe_key
   AWS_ACCESS_KEY_ID=your_aws_key
   AWS_SECRET_ACCESS_KEY=your_aws_secret
   AWS_REGION=us-east-1
   SNS_TOPIC_ARN=arn:aws:sns:us-east-1:123456789:appointments
   JWT_SECRET=your_jwt_secret
   EOF
   \`\`\`

3. **Start MongoDB**
   \`\`\`bash

   # If using local MongoDB

   mongod --dbpath /data/db
   \`\`\`

4. **Run backend**
   \`\`\`bash
   npm run dev

   # Server runs on <http://localhost:5000>

   \`\`\`

### Frontend Setup

1. **Navigate and install**
   \`\`\`bash
   cd ../frontend
   npm install
   \`\`\`

2. **Configure API endpoint**
   \`\`\`bash

   # Create .env

   echo "REACT_APP_API_URL=<http://localhost:5000/api>" > .env
   echo "REACT_APP_STRIPE_PUBLIC_KEY=pk_test_your_public_key" >> .env
   \`\`\`

3. **Run frontend**
   \`\`\`bash
   npm start

   # App runs on <http://localhost:3000>

   \`\`\`

## 🧠 Smart Slot Manager Algorithm

The core innovation of this system is the intelligent slot management:

\`\`\`javascript
// slot_manager.js
class SlotManager {
  constructor(business) {
    this.business = business;
    this.bufferTime = business.bufferTime || 15; // minutes
  }

  async getAvailableSlots(date, serviceId) {
    const service = await Service.findById(serviceId);
    const duration = service.duration;
    const businessHours = this.business.hours[date.getDay()];

    // Generate all possible slots
    const allSlots = this.generateTimeSlots(
      businessHours.start,
      businessHours.end,
      duration + this.bufferTime
    );
    
    // Get existing bookings for the date
    const bookings = await Appointment.find({
      businessId: this.business._id,
      date: date,
      status: { $ne: 'cancelled' }
    });
    
    // Filter out booked slots
    const availableSlots = allSlots.filter(slot => {
      return !this.hasConflict(slot, duration, bookings);
    });
    
    return availableSlots;
  }

  hasConflict(slot, duration, bookings) {
    const slotStart = new Date(slot);
    const slotEnd = new Date(slotStart.getTime() + duration * 60000);

    return bookings.some(booking => {
      const bookingStart = new Date(booking.startTime);
      const bookingEnd = new Date(booking.endTime);
      
      // Check for overlap
      return (slotStart < bookingEnd && slotEnd > bookingStart);
    });
  }

  generateTimeSlots(start, end, interval) {
    const slots = [];
    let current = new Date(\`2000-01-01T\${start}\`);
    const endTime = new Date(\`2000-01-01T\${end}\`);

    while (current < endTime) {
      slots.push(current.toTimeString().slice(0, 5));
      current = new Date(current.getTime() + interval * 60000);
    }
    
    return slots;
  }
}
\`\`\`

### Key Algorithm Features

- ✅ **Conflict Detection**: Prevents overlapping appointments
- ✅ **Buffer Time**: Adds gaps for preparation/cleanup
- ✅ **Business Hours**: Respects operating hours per day
- ✅ **Service Duration**: Accounts for different service lengths
- ✅ **Real-time Updates**: Reflects bookings immediately

## 💳 Payment Integration

### Stripe Checkout Flow

\`\`\`javascript
// payment_service.js
const createPaymentIntent = async (amount, customerId) => {
  const paymentIntent = await stripe.paymentIntents.create({
    amount: amount * 100, // Convert to cents
    currency: 'usd',
    customer: customerId,
    payment_method_types: ['card'],
    metadata: {
      type: 'appointment_deposit'
    }
  });
  
  return paymentIntent.client_secret;
};
\`\`\`

### Payment Features

- 💰 **Deposit System**: Require 20-50% upfront payment
- 🔄 **Refund Policy**: Automated refunds for cancellations
- 💳 **Saved Cards**: Store payment methods for repeat customers
- 📧 **Receipts**: Automatic email receipts via Stripe

## 📲 SMS Notification System

### AWS SNS Integration

\`\`\`javascript
// sms_service.js
const sendSMS = async (phoneNumber, message) => {
  const params = {
    Message: message,
    PhoneNumber: phoneNumber,
    MessageAttributes: {
      'AWS.SNS.SMS.SMSType': {
        DataType: 'String',
        StringValue: 'Transactional'
      }
    }
  };
  
  await sns.publish(params).promise();
};

// Notification templates
const templates = {
  confirmation: (name, date, time) =>
    \`Hi \${name}! Your appointment is confirmed for \${date} at \${time}. Reply CANCEL to cancel.\`,
  
  reminder: (name, hours) =>
    \`Hi \${name}! Reminder: Your appointment is in \${hours} hours. See you soon!\`,
  
  cancellation: (name) =>
    \`Hi \${name}, your appointment has been cancelled. Your refund will be processed in 5-7 days.\`
};
\`\`\`

### Notification Types

- ✅ **Booking Confirmation**: Sent immediately
- ⏰ **24-Hour Reminder**: Sent day before
- 🔔 **1-Hour Reminder**: Sent hour before
- ❌ **Cancellation Notice**: Sent on cancellation
- ⭐ **Feedback Request**: Sent after appointment

## 📊 Database Schema

### Appointment Model

\`\`\`javascript
const appointmentSchema = new Schema({
  businessId: { type: ObjectId, ref: 'Business', required: true },
  customerId: { type: ObjectId, ref: 'User', required: true },
  serviceId: { type: ObjectId, ref: 'Service', required: true },
  date: { type: Date, required: true },
  startTime: { type: Date, required: true },
  endTime: { type: Date, required: true },
  status: {
    type: String,
    enum: ['pending', 'confirmed', 'completed', 'cancelled'],
    default: 'pending'
  },
  payment: {
    amount: Number,
    status: String,
    stripePaymentId: String
  },
  notes: String,
  createdAt: { type: Date, default: Date.now }
});
\`\`\`

## 📈 Analytics Dashboard

### Key Metrics

- 📊 **Booking Rate**: Appointments per day/week/month
- 💰 **Revenue**: Total and average per appointment
- ❌ **No-Show Rate**: Percentage of missed appointments
- ⭐ **Customer Satisfaction**: Average rating from feedback
- 📅 **Peak Hours**: Most popular booking times

### Reports

- Daily booking summary
- Monthly revenue report
- Customer retention analysis
- Service popularity ranking

## 🔐 Security Features

- 🔒 **JWT Authentication**: Secure API access
- 🛡️ **Input Validation**: Prevent SQL injection and XSS
- 🔐 **Password Hashing**: bcrypt for user passwords
- 🌐 **CORS**: Configured for frontend domain only
- 📝 **Audit Logs**: Track all booking changes

## 🎯 Multi-Tenant Support

### Business Isolation

- Each business has separate data
- Custom branding per business
- Individual business hours and services
- Separate payment accounts

### Tenant Management

\`\`\`javascript
// Business schema
const businessSchema = new Schema({
  name: String,
  slug: { type: String, unique: true },
  logo: String,
  hours: {
    monday: { start: String, end: String },
    tuesday: { start: String, end: String },
    // ... other days
  },
  services: [{ type: ObjectId, ref: 'Service' }],
  stripeAccountId: String,
  settings: {
    bufferTime: Number,
    cancellationPolicy: String,
    depositPercentage: Number
  }
});
\`\`\`

## 🔄 Future Enhancements

- [ ] Video consultation integration (Zoom/Google Meet)
- [ ] Waitlist management
- [ ] Group bookings
- [ ] Loyalty program
- [ ] Mobile app (iOS/Android)
- [ ] Calendar sync (Google Calendar, Outlook)
- [ ] Multi-language support

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is open source and available under the MIT License.

## 👤 Author

**Abhinandan Sinha**

- Portfolio: [abhinandansinha.com](https://abhinandansinha01.github.io/portfolio-2026/portfolio/index.html)
- LinkedIn: [abhinandansinha01](https://linkedin.com/in/abhinandansinha01)
- GitHub: [@Abhinandansinha01](https://github.com/Abhinandansinha01)

---

<div align="center">
  <i>Scheduling made simple, one appointment at a time 📅</i>
</div>
