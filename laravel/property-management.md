**Property Management**

- **Image Handling**
  - drag and drop option to upload images 
  - Assign names or titles to images.
  - Order images to prioritize which ones are used first.

**Pricing**

- **Regular Rate**
  - The regular rate will be applied if no seasonal rate matches.
  - The rate is based on the booking date, not the creation date.

- **Seasonal Rate**
  - Offer seasonal rate options for specific occasions, such as Christmas, summer, winter, etc.
  - Charge different rates during these periods as applicable.

**Property Block Dates**

- Provide an option to specify a date range to block bookings for a property.

**Settings Page**

- Utilize JSON-based settings for configuration.

  - **General Settings**
    - Initially, include fields for notification email and sales tax.
    - Plan to add sales tax based on location in future updates.
    - Provide an option to change the label from "Owner" to "Agent" throughout the application.

  - **Payment Gateway Settings**
    - Allow payment gateway settings to be saved in the database similarly to other settings.

  - **Email Settings**
    - Enable users to save SMTP credentials in the database instead of storing them in the `.env` file.

**Booking Module for Admin**

- **Workflow:**
  1. **Find Property**
     - Locate the desired property and click the "Book" button to access the booking form.
  
  2. **Select Dates**
     - Display a calendar with available dates.
  
  3. **Customer Details**
     - Enter customer information.
     - Handle existing customers by ensuring customer IDs are unique rather than relying solely on email addresses.
  
  4. **Date Range and Rates**
     - Select a date range; the system will automatically determine the applicable rates.
  
  5. **Availability Check**
     - If selected dates are blocked, display an error message.
     - Availability is from 11 AM to 11 AM the next day.
     - For example, if a property is booked from the 21st to the 22nd, it becomes available for booking again on the 22nd at 11 AM.
  
  6. **Email Notifications**
     - Include checkboxes to send emails to the customer, agent, and other relevant parties as needed.

**Customer Email Contents**

- **Details to Include:**
  - Property name
  - Link to the property listing
  - Map link for the property location
  - Invoice detailing the subtotal, tax, and total amount

**Reservation Management**

- **Reservation List and View Page**
  - Display a list of all reservations with options to view details.

- **Cancellation and Refunds**
  - Provide an option to cancel a reservation.
  - Upon cancellation, initiate a refund request.
  - Include a dedicated page to handle and process refunds.
