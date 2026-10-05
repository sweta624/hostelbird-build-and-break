# hostelbird-build-and-break
Product audit, UX bug analysis, and production code fixes for the Hostelbird mobile app
**Submitted by:** Sweta Yadav  

---

1. Unhandled OTP Timeout Exception

Problem: Requesting an OTP displays a raw backend stack trace (Exception: Error exception OTP... TimeoutException after 0:00:15...) directly on the client UI.   

Impact: Exposing internal unhandled exceptions damages user trust and signals weak error handling during onboarding.   

Solution: Intercept API timeout exceptions in the repository layer and display a clean, user-friendly alert banner.   

Code Fix (Flutter):
Future verifyOtp(String phone, String otp) async {
try {
final response = await apiClient.post('/verify-otp', data: {'phone': phone, 'otp': otp});
} on TimeoutException catch () {
state = AuthState.error(
message: "SMS is taking longer than expected. Please check your network and retry in 60s."
);
} catch (e) {
state = AuthState.error(
message: "Unable to send verification code. Please try again."
);
}
}
2. Mandatory Authentication Gate

Problem: Users are forced to log in and provide personal details before browsing stays or searching destinations.

Impact: Causes high top-of-funnel drop-off for casual users who want to explore inventory before signing up.

Solution: Add a "Skip / Guest Mode" bypass option on the login screen, prompting authentication only at checkout.

Code Fix (Flutter):
Widget buildAuthHeader(BuildContext context) {
return Align(
alignment: Alignment.topRight,
child: TextButton(
onPressed: () {
UserSession.setGuestMode(true);
Navigator.pushReplacementNamed(context, '/home');
},
child: const Text("Skip for now ->"),
),
);
}
3. Unresponsive Search Input Rows

Problem: Tapping "Select Destination" or "Travellers" does nothing until the bottom "Next" button is clicked.   

Impact: Violates standard touch UI patterns where input rows are expected to be directly interactive.   

Solution: Attach explicit tap event listeners across full row container elements.

Code Fix (React Native):
const SearchForm = () => (

 openCityPickerModal()}>

Select destination

 openDatePickerModal()}>

Select dates


);
4. Bottom Navigation Bar Content Occlusion

Problem: On the "Choose your rooms" view, price breakdowns, taxes, and secondary action buttons are hidden behind the native device navigation bar.   

Impact: Prevents users from verifying final pricing or tapping checkout controls cleanly.   

Solution: Wrap sticky checkout bars in a SafeArea component to respect native window insets.   

Code Fix (Flutter):
Widget buildCheckoutFooter(BuildContext context) {
return SafeArea(
top: false,
child: Container(
padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 12.0),
child: Row(
mainAxisAlignment: MainAxisAlignment.spaceBetween,
children: [
PriceSummaryWidget(total: 31752.00, taxesIncluded: true),
ElevatedButton(
onPressed: () => navigateToCheckout(),
child: const Text("CHECKOUT ->"),
),
],
),
),
);
}
5. Messaging Service Fallback Handling

Problem: Tapping the messaging tab triggers generic system errors when backend sockets disconnect or degrade.   

Impact: Leaves travelers stranded with no direct communication or support route.   

Solution: Implement a graceful fallback screen directing users to WhatsApp support when chat servers fail.   

Code Fix (Flutter):
Widget buildMessagesScreen() {
return Center(
child: Column(
mainAxisAlignment: MainAxisAlignment.center,
children: [
const Icon(Icons.support_agent, size: 64, color: Colors.grey),
const SizedBox(height: 12),
const Text("In-App Chat Unavailable", style: TextStyle(fontWeight: FontWeight.bold)),
const SizedBox(height: 16),
ElevatedButton.icon(
onPressed: () => launchWhatsAppSupport(),
icon: const Icon(Icons.chat_bubble_outline),
label: const Text("Chat on WhatsApp Support"),
),
],
),
);
}
