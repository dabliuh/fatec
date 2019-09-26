# fatec
App Notas Diárias - Swift 5.1

# AppDelegate.swift
//
//  AppDelegate.swift
//  Notas Diarias
//
//  Created by WILLIAN SOUZA LIMA on 25/09/2019.
//  Copyright © 2019 Willian Souza Lima. All rights reserved.
//

import UIKit
import CoreData //Responsável por salvar e persistir dados no banco

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?


    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        return true
    }

    func applicationWillResignActive(_ application: UIApplication) {
        // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
        // Use this method to pause ongoing tasks, disable timers, and invalidate graphics rendering callbacks. Games should use this method to pause the game.
    }

    func applicationDidEnterBackground(_ application: UIApplication) {
        // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
        // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    }

    func applicationWillEnterForeground(_ application: UIApplication) {
        // Called as part of the transition from the background to the active state; here you can undo many of the changes made on entering the background.
    }

    func applicationDidBecomeActive(_ application: UIApplication) {
        // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    }

    func applicationWillTerminate(_ application: UIApplication) {
        // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
        // Saves changes in the application's managed object context before the application terminates.
        self.saveContext()
    }

    // MARK: - Core Data stack

    lazy var persistentContainer: NSPersistentContainer = {
        /*
         The persistent container for the application. This implementation
         creates and returns a container, having loaded the store for the
         application to it. This property is optional since there are legitimate
         error conditions that could cause the creation of the store to fail.
        */
        let container = NSPersistentContainer(name: "Notas_Diarias")
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                // Replace this implementation with code to handle the error appropriately.
                // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
                 
                /*
                 Typical reasons for an error here include:
                 * The parent directory does not exist, cannot be created, or disallows writing.
                 * The persistent store is not accessible, due to permissions or data protection when the device is locked.
                 * The device is out of space.
                 * The store could not be migrated to the current model version.
                 Check the error message to determine what the actual problem was.
                 */
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        return container
    }()

    // MARK: - Core Data Saving support

    func saveContext () {
        let context = persistentContainer.viewContext
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                // Replace this implementation with code to handle the error appropriately.
                // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
                let nserror = error as NSError
                fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
            }
        }
    }

}

# ListarAnotacoesTableViewController.swift
//
//  ListarAnotacoesTableViewController.swift
//  Notas Diarias
//
//  Created by WILLIAN SOUZA LIMA on 25/09/2019.
//  Copyright © 2019 Willian Souza Lima. All rights reserved.
//

import UIKit
import CoreData

class ListarAnotacoesTableViewController: UITableViewController {

    var context: NSManagedObjectContext!
    var anotacoes: [NSManagedObject] = [] //array para guardar anotaçoes
    
    override func viewDidLoad() {
        super.viewDidLoad()

        let appDelegate = UIApplication.shared.delegate as! AppDelegate
        context = appDelegate.persistentContainer.viewContext     
        
    }
    
    override func viewDidAppear(_ animated: Bool) {
        self.recuperarAnotacoes()
    }
    
    func recuperarAnotacoes() {
        let requisicao = NSFetchRequest<NSFetchRequestResult>(entityName: "Anotacao")
        let ordenacao = NSSortDescriptor(key: "data", ascending: false)
        requisicao.sortDescriptors = [ordenacao]
        
        do {
            let anotacoesRecuperadas = try context.fetch(requisicao)
            self.anotacoes = anotacoesRecuperadas as! [NSManagedObject]
            self.tableView.reloadData()
            
        } catch let erro as Error {
            print("Erro ao recuperar anotações: \(erro.localizedDescription)")
            
        }
    }

    // MARK: - Table view data source

    override func numberOfSections(in tableView: UITableView) -> Int {
        // #warning Incomplete implementation, return the number of sections
        return 1
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        // #warning Incomplete implementation, return the number of rows
        return self.anotacoes.count
    }

    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        self.tableView.deselectRow(at: indexPath, animated: true)
        
        let indice = indexPath.row
        let anotacao = self.anotacoes[indice]
        self.performSegue(withIdentifier: "verAnotacao", sender: anotacao)
        
    }
    
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if segue.identifier == "verAnotacao" {
            let viewDestino = segue.destination as! AnotacaoViewController
            viewDestino.anotacao = sender as? NSManagedObject
            
        }
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let celula = tableView.dequeueReusableCell(withIdentifier: "celula", for: indexPath)

        // Configure the cell...
        let anotacao = self.anotacoes[ indexPath.row ]
        let textoRecuperado = anotacao.value(forKey: "texto") //texto da anotacao recuperada
        let dataRecuperada = anotacao.value(forKey: "data") //exibe a data na nota
        
        //formatar data
        let dateFormatter = DateFormatter()
        dateFormatter.dateFormat = "dd/MM/yyyy hh:mm"
        
        let novaData = dateFormatter.string(from: dataRecuperada as! Date)
        
        celula.textLabel?.text = textoRecuperado as? String
        celula.detailTextLabel?.text = novaData

        
        return celula
    }
    
    //deletando nota
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        
        if editingStyle == UITableViewCell.EditingStyle.delete {
            
            let indice = indexPath.row
            let anotacao = self.anotacoes[indice]
            
            self.context.delete(anotacao)
            self.anotacoes.remove(at: indice)
            
            self.tableView.deleteRows(at: [indexPath], with: .automatic)
            
            do{
                try self.context.save()
            }catch let erro {
                print("Erro ao remover item \(erro)")
            }
        }
    }
    

    /*
    // Override to support conditional editing of the table view.
    override func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
        // Return false if you do not want the specified item to be editable.
        return true
    }
    */

    /*
    // Override to support editing the table view.
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            // Delete the row from the data source
            tableView.deleteRows(at: [indexPath], with: .fade)
        } else if editingStyle == .insert {
            // Create a new instance of the appropriate class, insert it into the array, and add a new row to the table view
        }    
    }
    */

    /*
    // Override to support rearranging the table view.
    override func tableView(_ tableView: UITableView, moveRowAt fromIndexPath: IndexPath, to: IndexPath) {

    }
    */

    /*
    // Override to support conditional rearranging of the table view.
    override func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool {
        // Return false if you do not want the item to be re-orderable.
        return true
    }
    */

    /*
    // MARK: - Navigation

    // In a storyboard-based application, you will often want to do a little preparation before navigation
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        // Get the new view controller using segue.destination.
        // Pass the selected object to the new view controller.
    }
    */

}


# AnotacaoViewController.swift
//
//  AnotacaoViewController.swift
//  Notas Diarias
//
//  Created by WILLIAN SOUZA LIMA on 25/09/2019.
//  Copyright © 2019 Willian Souza Lima. All rights reserved.
//

import UIKit
import CoreData

class AnotacaoViewController: UIViewController {

    @IBOutlet weak var texto: UITextView!
    var context: NSManagedObjectContext!
    var anotacao: NSManagedObject!
    
    
    // Do any additional setup after loading the view.
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //configuracoes iniciais
        self.texto.becomeFirstResponder() //esse texto será o primeiro a usar o teclado
        if anotacao != nil { //atualizar
            if let textoRecuperado = anotacao.value(forKey: "texto"){
                self.texto.text = String(describing: textoRecuperado)
            }
            
        }else{
            self.texto.text = ""
            
        }
        
        
        //deixa a caixa de texto vazia
        
        
        let appDelegate = UIApplication.shared.delegate as! AppDelegate
        context = appDelegate.persistentContainer.viewContext
        //context é o nosso gerenciador, responsavel pela persistencia dos dados

    }
    
    @IBAction func salvar(_ sender: Any) {
        if anotacao != nil { //atualizar
            self.atualizarAnotacao()
        }else{
            self.salvarAnotacao() //cria um novo ou atualiza um item existente
        }
        
        
        //retorna para a tela inicial
        self.navigationController?.popToRootViewController(animated: true)
    }
    
    //cria o metodo atualizar Anotação
    func atualizarAnotacao() {
        anotacao.setValue( self.texto.text, forKey: "texto")
        anotacao.setValue( Date(), forKey: "data")
        
        do {
            try context.save()
            print("Sucesso ao atualizar anotação")
        } catch let erro as Error {
            print("Erro ao atualizar anotação: \(erro.localizedDescription) ")
        }
        
    }
    
    //cria o metodo salvar Anotação
    func salvarAnotacao() {
        
        //Cria objeto para anotação
        let novaAnotacao = NSEntityDescription.insertNewObject(forEntityName: "Anotacao", into: context)
        
        //configura anotação
        novaAnotacao.setValue( self.texto.text , forKey: "texto")
        novaAnotacao.setValue( Date() , forKey: "data")
        
        do {
            try context.save()
            print("Sucesso ao salvar anotação")
        } catch let erro as Error {
            print("Erro ao salvar anotação: \(erro.localizedDescription) ")
        }
        
    }
    
    
    
    
    /*
    // MARK: - Navigation

    // In a storyboard-based application, you will often want to do a little preparation before navigation
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        // Get the new view controller using segue.destination.
        // Pass the selected object to the new view controller.
    }
    */

}
