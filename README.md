import cx_Oracle

################Database class
class Database:
    dns = cx_Oracle.makedsn("127.0.0.1",1521,"XE")
    con = cx_Oracle.connect("system","admin",dns)
    cur = con.cursor()
    def addCustomer(self,name,DOB,address,phone,password):
        querystring = "insert into TESTDB.CUSTOMER(id,acc_no,name,dob,address,phone,password,balance) values (TESTDB.ID_SEQUENCE.NEXTVAL,TESTDB.ACC_SEQUENCE.NEXTVAL,'"+name+"',TO_DATE('"+DOB+"','yyyy/mm/dd'),'"+address+"',"+phone+",'"+password+"',500)"  
        Database.cur.execute(querystring)
        Database.cur.execute("select id,acc_no from TESTDB.CUSTOMER where name='"+name+"'")
        print"\nSuccessful sign-up\n your customer id and account number are:"
        res = Database.cur.fetchall()
        for n in res:
            print(n)
        Database.con.commit()
        
    def addrChange(self,username,newaddr):
        querystring="update TESTDB.CUSTOMER set address='"+newaddr+"' where name='"+username+"'"
        Database.cur.execute(querystring)
        Database.con.commit()
        print"\nAddress successfully changed\n"

    def deposit(self,username):
       amt=raw_input("Enter the amount:")
       prompt=int(raw_input("""PRESS 1 To deposit in Savings account\n"""+
                            """PRESS 2 To deposit in current account\n"""+
                            """PRESS 3 To deposit in FD account\n"""))
       if prompt==1:
           querystring="update TESTDB.SACC set sbalance=sbalance+"+amt+" where name='"+username+"'"
           Database.cur.execute(querystring)
           Database.con.commit()
           print"\nsuccessfully credited in savings account\n"
       elif prompt==2:
           querystring="update TESTDB.CACC set cbalance=cbalance+"+amt+" where name='"+username+"'"
           Database.cur.execute(querystring)
           Database.con.commit()
           print"\nsuccessfully credited in Current account\n"
       elif prompt==3:
           querystring="update TESTDB.FDCC set fbalance=fbalance+"+amt+" where name='"+username+"'"
           Database.cur.execute(querystring)
           Database.con.commit()
           print"\nsuccessfully credited in FD account\n"
       else:
           print "You have pressed the wrong key, please try again"
           s=Database()
           s.deposit()
            

    def withdraw(self,username):
        amt=raw_input("Enter the amount:")
        prompt=int(raw_input("""PRESS 1 To withdraw from Savings account\n"""+
                             """PRESS 2 To withdraw from current account\n"""))
        if prompt==1:
            querysbalance="select sbalance from TESTDB.SACC where name='"+username+"'"
            Database.cur.execute(querysbalance)
            res = Database.cur.fetchall()
            for n in res:
                if n>int(amt):
                    count=1
            if count==1:
                querystring1="update TESTDB.SACC set sbalance=sbalance-"+amt+" where name='"+username+"'"
                Database.cur.execute(querystring1)
                Database.con.commit()
                print"\nsuccessfully debited\n"
            else:
                print"\ninsufficient balance\n"
        elif prompt==2:
            querycbalance="select cbalance from TESTDB.CACC where name='"+username+"'"
            Database.cur.execute(querycbalance)
            res = Database.cur.fetchall()
            for n in res:
                if n>int(amt):
                    count=1
            if count==1:
                querystring2="update TESTDB.CACC set cbalance=cbalance-"+amt+" where name='"+username+"'"
                Database.cur.execute(querystring2)
                Database.con.commit()
                print"\nsuccessfully debited\n"
            else:
                print"\ninsufficient balance\n"

    def transfer(self,username):
        acc=raw_input("enter the account number of recepient:")
        amt=raw_input("enter the amount:")
        querybalance="select balance from TESTDB.CUSTOMER where name='"+username+"'"
        Database.cur.execute(querybalance)
        res = Database.cur.fetchall()
        for n in res:
            if n>int(amt):
                count=1
        if count==1:
            querystring1="update TESTDB.CUSTOMER set balance=balance-"+amt+" where name='"+username+"'"
            querystring2="update TESTDB.CUSTOMER set balance=balance+"+amt+" where acc_no="+acc+""
            Database.cur.execute(querystring1)
            Database.cur.execute(querystring2)
            Database.con.commit()
            print"\nsuccessfully transfered\n"
        else:
            print"\ninsufficient balance\n"
            
        
    def statement(self,username):
         querystring1="select name from TESTDB.CUSTOMER where name='"+username+"'"
         Database.cur.execute(querystring1)
         Database.con.commit()
         res = Database.cur.fetchall()
         for n in res:
             print(n)
         querystring2="select sacc_no,sbalance from TESTDB.SACC where name='"+username+"'"
         Database.cur.execute(querystring2)
         Database.con.commit()
         res = Database.cur.fetchall()
         for n in res:
             if res:
                 print(n)
             else:
                 print"N/A\n"
         querystring3="select cacc_no,cbalance from TESTDB.CACC where name='"+username+"'"
         Database.cur.execute(querystring3)
         Database.con.commit()
         res = Database.cur.fetchall()
         for n in res:
             if res:
                 print(n)
             else:
                 print"N/A\n"
         querystring4="select fdcc_no,fbalance from TESTDB.FDCC where name='"+username+"'"
         Database.cur.execute(querystring4)
         Database.con.commit()
         res = Database.cur.fetchall()
         for n in res:
             if res:
                 print(n)
             else:
                 print"N/A\n"

    def closeacc(self,username):
        querystring1="select acc_no,id,name from TESTDB.CUSTOMER where name='"+username+"'"
        Database.cur.execute(querystring1)
        res = Database.cur.fetchall()
        for n in res:
             a=str(n[0])
             b=str(n[1])
             c=str(n[2])
        querystring2="insert into TESTDB.CLOSE values("+a+","+b+",'"+c+"')"
        querystring3="delete from TESTDB.CUSTOMER where name='"+username+"'"
        Database.cur.execute(querystring2)
        Database.cur.execute(querystring3)
        Database.con.commit()
        print"\nAccount closed\n"

    def adminMenu(self):
        dbobj=Database()
        prompt=int(raw_input("""PRESS 1 To see history of closed accounts\n"""+                   
                             """PRESS 2 To see FD report\n"""+
                             """PRESS 3 To see FD report with particular amount\n"""+                   
                             """PRESS 4 To see loan report\n"""+
                             """PRESS 5 To see loan report with particular amount\n"""+                   
                             """PRESS 6 To see loan - FD report of customers \n"""+
                             """PRESS 7 To see Report of Customers who are yet to avail a loan \n"""+                   
                             """PRESS 8 To see Report of Customers who are yet to open a FD account\n"""+
                             """PRESS 9 To see Report of Customers who neither have a loan nor a FD account with the bank\n"""+                   
                             """PRESS 10 To logout\n"""))
                             
        if prompt==1:
            querystring="select * from TESTDB.CLOSE"
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==2:
            querystring="select * from TESTDB.FDCC"
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==3:
            amount=(raw_input("""Enter the amount:\n"""))
            querystring="select * from TESTDB.FDCC where fbalance>="+amount+""
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==4:
            querystring="select * from TESTDB.LOAN"
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==5:
            amount=(raw_input("""Enter the amount\n"""))
            querystring="select * from TESTDB.LOAN where loan_amt>="+amount+""
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==6:
            querystring="SELECT l.id,l.name FROM testdb.loan l inner join testdb.fdcc fd on l.id=fd.id WHERE l.loan_amt >= fd.fbalance"
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==7:
            querystring="SELECT id,name FROM testdb.customer WHERE id NOT IN (SELECT DISTINCT id FROM testdb.loan)"
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==8:
            querystring="SELECT id,name FROM testdb.customer WHERE id NOT IN (SELECT DISTINCT id FROM testdb.fdcc)"
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==9:
            querystring="select cus.id,cus.name from testdb.customer cus where not exists (select f.id from testdb.fdcc f  where cus.id = f.id) and not exists (select l.id from testdb.loan l where cus.id = l.id)"
            Database.cur.execute(querystring)
            Database.con.commit()
            res = Database.cur.fetchall()
            for n in res:
                print(n)
            dbobj.adminMenu()
        elif prompt==10:
            print"\nyou are logged out\n"
        else:
            print"\nchoose a valid option\n"

    def SA_acc(self,username):
        querystring1="select id from TESTDB.CUSTOMER where name='"+username+"'"
        Database.cur.execute(querystring1)
        res1 = Database.cur.fetchall()
        for n in res1:
            id=str(n[0])  
        querystring2="insert into TESTDB.SACC values("+id+",TESTDB.SA_SEQUENCE.NEXTVAL,'"+username+"',500)"
        Database.cur.execute(querystring2)
        Database.con.commit()
        print"\nSavings Account created successfully\nyour account details are as follows:"
        querystring3="select sacc_no,name,sbalance from TESTDB.SACC where name='"+username+"'"
        Database.cur.execute(querystring3)
        res2 = Database.cur.fetchall()
        for n in res2:
            print(n)
    def CA_acc(self,username):
        querystring1="select id from TESTDB.CUSTOMER where name='"+username+"'"
        Database.cur.execute(querystring1)
        res1 = Database.cur.fetchall()
        for n in res1:
            id=str(n[0])  
        querystring2="insert into TESTDB.CACC values("+id+",TESTDB.CA_SEQUENCE.NEXTVAL,'"+username+"',500)"
        Database.cur.execute(querystring2)
        Database.con.commit()
        print"\nCurrent Account created successfully\nyour account details are as follows:"
        querystring3="select cacc_no,name,cbalance from TESTDB.CACC where name='"+username+"'"
        Database.cur.execute(querystring3)
        res2 = Database.cur.fetchall()
        for n in res2:
            print(n)
    def FD_acc(self,username):
        querystring1="select id from TESTDB.CUSTOMER where name='"+username+"'"
        Database.cur.execute(querystring1)
        res1 = Database.cur.fetchall()
        for n in res1:
            id=str(n[0])  
        querystring2="insert into TESTDB.FDCC values("+id+",TESTDB.FD_SEQUENCE.NEXTVAL,'"+username+"',1000)"
        Database.cur.execute(querystring2)
        Database.con.commit()
        print"\nFD Account created successfully\nyour account details are as follows:"
        querystring3="select fdcc_no,name,fbalance from TESTDB.FDCC where name='"+username+"'"
        Database.cur.execute(querystring3)
        res2 = Database.cur.fetchall()
        for n in res2:
            print(n)
    def availLoan(self,username):
        querystring1="select id,sbalance from TESTDB.SACC where name='"+username+"'"
        Database.cur.execute(querystring1)
        res1 = Database.cur.fetchall()
        for n in res1:
            cid=str(n[0])
            bal=str(2*n[1])
        querystring2="insert into TESTDB.LOAN values("+cid+",TESTDB.LOAN_SEQUENCE.NEXTVAL,"+bal+",1,'"+username+"')"
        Database.cur.execute(querystring2)
        Database.con.commit()
        querystring3="select * from TESTDB.LOAN where id='"+cid+"'"
        Database.cur.execute(querystring3)
        print"Loaan details are as follows:\n"
        res2 = Database.cur.fetchall()
        for n in res2:
            print(n)
        

############################################################

class UserOper(Database):
    def register(self):
        print"Enter the following details\n"
        name=raw_input("Customer name:")
        DOB=raw_input("Date of birth:")
        address=raw_input("Address:")
        phone=raw_input("Phone-no:")
        password=raw_input("hint:Password must be atlest 6 characters\nPassword:")
        obj = Database()
        obj.addCustomer(name,DOB,address,phone,password)

    def newAccount(self,username):
        prompt=int(raw_input("""PRESS 1 To open Savings account\n"""+
                             """PRESS 2 To open current account\n"""+
                             """PRESS 3 To open FD account\n"""))
        dbobj=Database()
        if prompt==1:
            print("SA account creation\n")
            dbobj.SA_acc(username)
        elif prompt==2:                                                                     
            print("CA account creation\n")
            dbobj.CA_acc(username)
        elif prompt==3:                                                                     
            print("FD account creation\n")
            dbobj.FD_acc(username)
        else:
            print "You have pressed the wrong key, please try again"
            s=UserOper()
            s.newAccount()                    
        
    def subMenu(self,username,password):
            print ("Welcome to PostBank, We care for you\n")                                    
            prompt=int(raw_input("""PRESS 1 To Address Change\n"""+
                                 """PRESS 2 To Open new account\n"""+
                                 """PRESS 3 To Deposit\n"""+
                                 """PRESS 4 To Withdraw\n"""+
                                 """PRESS 5 To Transfer Money\n"""+
                                 """PRESS 6 To Print Statement\n"""+
                                 """PRESS 7 To close account\n"""+
                                 """PRESS 8 To avail loan\n"""+
                                 """PRESS 9 To logout\n"""))
            dbobj=Database()
            if prompt==1:
                print("Address Change\n")
                newaddr=raw_input("Enter the new address:")
                dbobj.addrChange(username,newaddr)
            elif prompt==2:                                                                     
                s=UserOper()
                s.newAccount(username)
                s.subMenu(username,password)
            elif prompt==3:                                                                     
                dbobj.deposit(username)
                s=UserOper()
                s.subMenu(username,password)
            elif prompt==4:                                                                     
                dbobj.withdraw(username)
                s=UserOper()
                s.subMenu(username,password)
            elif prompt==5:                                                                     
                dbobj.transfer(username)
                s=UserOper()
                s.subMenu(username,password)
            elif prompt==6:                                                                     
                dbobj.statement(username)
                s=UserOper()
                s.subMenu(username,password)
            elif prompt==7:                                                                     
                dbobj.closeacc(username)
                s=UserOper()
                s.subMenu(username,password)
            elif prompt==8:                                                                     
                dbobj.availLoan(username)
                s=UserOper()
                s.subMenu(username,password)
            elif prompt==9:                                                                     
                print"you are logged out"
                s=UserOper()
                s.signIn()
            else:                                                                               
                print "You have pressed the wrong key, please try again"
                s=UserOper()
                s.subMenu(username,password)
            
    def signIn(self):
        print("____________SIGN  IN_________________\n")
        username = raw_input("Username:")
        password = raw_input("Password:")
        querystring="select name,password from testdb.customer where name='"+username+"' and password='"+password+"'"
        Database.cur.execute(querystring)
        res = Database.cur.fetchall()
        print(res)
        if res:
            cus=UserOper()
            print"You are logged in"
            cus.subMenu(username,password)
        else:
            print"Invalid Username or Password"
            s=UserOper()
            s.signIn()
        

#################

class Admin(Database):
    def aSignIn(self):
        print("____________ADMIN SIGN  IN_________________\n")
        username = raw_input("Username:")
        password = raw_input("Password:")
        querystring="select uname,password from testdb.admin where uname='"+username+"' and password='"+password+"'"
        Database.cur.execute(querystring)
        res = Database.cur.fetchall()
        print(res)
        if res:
            hsc=Database()
            print"You are logged in"
            hsc.adminMenu()
            
        else:
            print"Invalid Username or Password"
            a=Admin()
            a.aSignIn()        
        

##############Main class#####

class Main:
    def mainMenu():
        print ("Welcome to PostBank, We care for you\n")                                    
        prompt=int(raw_input("""PRESS 1 To Sign Up\n"""+                   
                             """PRESS 2 To Sign In\n"""+
                             """PRESS 3 For Admin Sign In\n"""+
                             """PRESS 4 To Quit\n"""))
        cus=UserOper()#creates a new customer profile
        if prompt==1:
            cus.register()
        elif prompt==2:
            cus.signIn()
        elif prompt==3:
            adm=Admin()
            adm.aSignIn()
        elif prompt==4:                                                                     
            print(".......Thankyou........")    
        else:                                                                               
            print "You have pressed the wrong key, please try again"                         
    mainMenu()                                                 
###########################################################################################







                   
               
               
               
        
        
