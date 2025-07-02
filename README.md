# lydia256
import 'dart:math';
import 'package:flutter/material.dart';

void main() {
  runApp(SaccoVerificationApp());
}

class SaccoVerificationApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SACCO OTP Verification',
      theme: ThemeData(primarySwatch: Colors.green),
      home: PhoneNumberScreen(),
    );
  }
}

// Global OTP storage (simulating a temporary store)
Map<String, String> otpStorage = {};

class PhoneNumberScreen extends StatefulWidget {
  @override
  _PhoneNumberScreenState createState() => _PhoneNumberScreenState();
}

class _PhoneNumberScreenState extends State<PhoneNumberScreen> {
  final _phoneController = TextEditingController();

  String generateOTP() {
    final random = Random();
    return (1000 + random.nextInt(9000)).toString(); // 4-digit OTP
  }

  void _sendOTP() {
    String phone = _phoneController.text.trim();
    if (phone.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(
        content: Text("Please enter a phone number"),
      ));
      return;
    }

    String otp = generateOTP();
    otpStorage[phone] = otp;

    // For demo: show the OTP using a dialog (in real app you'd send SMS)
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text("OTP Sent"),
        content: Text("OTP for $phone is $otp"),
        actions: [
          TextButton(
            child: Text("Continue"),
            onPressed: () {
              Navigator.pop(context);
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => OTPScreen(phoneNumber: phone),
                ),
              );
            },
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("SACCO Login")),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          children: [
            Text("Enter your phone number", style: TextStyle(fontSize: 18)),
            SizedBox(height: 10),
            TextField(
              controller: _phoneController,
              keyboardType: TextInputType.phone,
              decoration: InputDecoration(
                hintText: "e.g. 0701234567",
                border: OutlineInputBorder(),
              ),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: _sendOTP,
              child: Text("Send OTP"),
            ),
          ],
        ),
      ),
    );
  }
}

class OTPScreen extends StatefulWidget {
  final String phoneNumber;

  OTPScreen({required this.phoneNumber});

  @override
  _OTPScreenState createState() => _OTPScreenState();
}

class _OTPScreenState extends State<OTPScreen> {
  final _otpController = TextEditingController();

  void _verifyOTP() {
    String enteredOTP = _otpController.text.trim();
    String? correctOTP = otpStorage[widget.phoneNumber];

    if (enteredOTP == correctOTP) {
      showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: Text("Success"),
          content: Text("Phone number verified successfully!"),
          actions: [
            TextButton(
              child: Text("OK"),
              onPressed: () {
                otpStorage.remove(widget.phoneNumber); // clear OTP
                Navigator.popUntil(context, (route) => route.isFirst);
              },
            ),
          ],
        ),
      );
    } else {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(
        content: Text("Invalid OTP. Try again."),
      ));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Verify OTP")),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          children: [
            Text("Enter the OTP sent to ${widget.phoneNumber}",
                style: TextStyle(fontSize: 18)),
            SizedBox(height: 10),
            TextField(
              controller: _otpController,
              keyboardType: TextInputType.number,
              maxLength: 4,
              decoration: InputDecoration(
                hintText: "Enter 4-digit OTP",
                border: OutlineInputBorder(),
              ),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: _verifyOTP,
              child: Text("Verify"),
            ),
          ],
        ),
      ),
    );
  }
}
