# Adapty

Step 1: 
Class Appdelegate


private func setupSDKs() {

PurchaseService.shared.setupIAP()

AppUtility.lockOrientation(.portrait, andRotateTo: .portrait)

}
