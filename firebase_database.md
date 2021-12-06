# firebase database

### 1
```
firebase 콘솔에 App 번들 및 database 추가
```

### 2. AppDelegate
pod file 에 firebase-database 추가후 기본세팅
``` swift
// didFinishLaunchingWithOption 메소드에 추가
FirebaseApp.Configure()
```

### 4. 사용예시

``` swift
var ref:DatabaseReference!
.
.
.
override func viewDidLoad(){
  super.viewDidLoad()
  ref = Database.database().reference()
  
  ref.observe(.value){snapshot in
            guard let value = snapshot.value as? [String: [String:Any]] else {return}
            
            do{
                let jsonData = try JSONSerialization.data(withJSONObject: value)
                let cardData = try JSONDecoder().decode([String:CreditCard].self, from: jsonData)
                let cardList = Array(cardData.values)
                self.creditCardList = cardList.sorted{$0.rank < $1.rank}
                
                DispatchQueue.main.async {
                    self.tableView.reloadData()
                }
            }catch let error{
                print("Error Json parsing \(error.localizedDescription)")
            }
        }
}
.
.
.
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        //
        let storyboard = UIStoryboard(name: "Main", bundle: Bundle.main)
        guard let detailViewController = storyboard.instantiateViewController(withIdentifier: "CardDetailViewController") as? CardDetailViewController else {return}
        
        detailViewController.promotionDetail = creditCardList[indexPath.row].promotionDetail
        self.show(detailViewController, sender: nil)
        
        
//        //Option1
        let cardID = creditCardList[indexPath.row].id
//        ref.child("Item\(cardID)isSelected").setValue(true)

//        //Option2        
        ref.queryOrdered(byChild: "id").queryEqual(toValue: cardID).observe(.value) { [weak self] snapshot in
            guard let self = self,
                  let value = snapshot.value as? [String:[String:Any]],
                  let key = value.keys.first else {return}
            
            self.ref.child("\(key)isSelected").setValue(true)
        }
    }
    
    override func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
        return true
    }
    
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete{
            //Option1
            let cardID = creditCardList[indexPath.row].id
            ref.child("Item\(cardID)").removeValue()
            
            //Option2
            ref.queryOrdered(byChild: "id").queryEqual(toValue: cardID).observe(.value) { [weak self] snapshot in
                guard let self = self,
                      let value = snapshot.value as? [String:[String:Any]],
                      let key = value.keys.first else {return}
                self.ref.child(key).removeValue()
            }
        }
    }



```

