# DEP.Y9
Purchase management



![Design preview for the Indent for Purchase page coding challenge](./Dashbord.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-21-52.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-26-57.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-26-57.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-27-20.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-29-18.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-34-18.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-38-29.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-39-25.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-50-41.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-15_23-54-17.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-16_00-00-23.png)
![Design preview for the Indent for Purchase page coding challenge](./image_2022-05-16_00-01-47.png)



from optparse import Values
import re
from flask import Flask, render_template, request, redirect, url_for, session, Response
import pyrebase
from datetime import date
from flask_session import Session
import pandas as pd
import gspread
import pandas as pd
from oauth2client.service_account import ServiceAccountCredentials
import random
# Python code to illustrate Sending mail with attachments
# from your Gmail account
# libraries to be imported
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

# define the scope
scope = ['https://spreadsheets.google.com/feeds','https://www.googleapis.com/auth/drive']
# add credentials to the account
creds = ServiceAccountCredentials.from_json_keyfile_name("cred.json", scope)
# authorize the clientsheet 
client = gspread.authorize(creds)
# get the instance of the Spreadsheet
sheet = client.open('sheet database')
# get the first sheet of the Spreadsheet
sheet_instance = sheet.get_worksheet(2)
# get all the records of the data
records_data = sheet_instance.get_all_records()
# convert the json to dataframe
df = pd.DataFrame.from_dict(records_data)

config = {
    "apiKey": "AIzaSyDrdiYaeeC_kFwgiR27apzWhX-dzoacOCI",
    "authDomain": "indent-for-purchase.firebaseapp.com",
    "projectId": "indent-for-purchase",
    "storageBucket": "indent-for-purchase.appspot.com",
    "messagingSenderId": "655214437659",
    "appId": "1:655214437659:web:61c24f217752a9358a1fbe",
    "measurementId": "G-5JPWPNQ52F",
    "databaseURL": "https://indent-for-purchase-default-rtdb.asia-southeast1.firebasedatabase.app/"
}


app = Flask(__name__)
firebase = pyrebase.initialize_app(config)          # connecting to firebase
db = firebase.database()
app.config["SESSION_PERMANENT"] = False
app.config["SESSION_TYPE"] = "filesystem"
Session(app)
@app.route('/myapplications/<username>')
def myapplications(username):                       # Function shows Indenter Application
    if(not session.get('login_check')):             # If Else condtion are used for gettong conditions
        return redirect(url_for('login'))           # dic dictionary stores the profile data
    fusername=sheet_instance.find(username)
    dic={}                                          
    dic['username']=sheet_instance.acell('B'+str(fusername.row)).value
    dic['id']=sheet_instance.acell('A'+str(fusername.row)).value
    dic['password']=sheet_instance.acell('C'+str(fusername.row)).value
    dic['email']=sheet_instance.acell('D'+str(fusername.row)).value
    L=int(sheet_instance.acell('E'+str(fusername.row)).value)
    dic['level']='INDENTER'
    todo = db.child("form").get()
    jsondata = todo.val()
    mydic = []                                      # Stores data of all application at one place, is a 2D matrix
    j = 1
    for i in jsondata.items():
        mydic1=[]                                       # Stores data of individual, is a 1D matrix
        # print(i[1]['user'])
        if(i[1]["user"]==username):
            mydic1.append(j)
            mydic1.append(i[0])
            mydic1.append(i[1]["date"])
            if(i[1]["statusA1"]=="false" and i[1]["statusR1"]=="false"):
                mydic1.append("Pending by supredent")
            elif(i[1]["statusR1"]=="true"):
                mydic1.append("Rejected by supredent")
            elif(i[1]["statusR2"]=="false" and i[1]["statusA2"]=="false"):
                mydic1.append("Pending by AR/DR/JR")
            elif(i[1]["statusR2"]=="true"):
                mydic1.append("Rejected by AR/DR/JR")
            elif(i[1]["statusR3"]=="false" and i[1]["statusA3"]=="false"):
                mydic1.append("Pending by registar")
            elif(i[1]["statusR3"]=="true"):
                mydic1.append("Rejected by registar")
            elif(i[1]["statusR4"]=="false" and i[1]["statusA4"]=="false"):
                mydic1.append("Pending by HOD")
            elif(i[1]["statusR4"]=="true"):
                mydic1.append("Rejected by HOD")
            elif(i[1]["statusR4"]=="false" and i[1]["statusA4"]=="true"):
                mydic1.append("Your Application is approved")
            mydic.append(mydic1)
            j=j+1
    return render_template("IndenterDashboard.html", parent_list=mydic,user=username,dic=dic) 

@app.route('/approversP1/<username>')
def approversP1(username):                          # Approver P1, P2, P3, and P4 are routes for displaying all the information about Respective approvers.
    if(not session.get('login_check')):             # checking if user logged-in or not
        return redirect(url_for('login'))
    try:
        todo = db.child("form").get()
        jsondata = todo.val()
        mydic = []                                  # Stores pending applications, 
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA1"]=="false" and i[1]["statusR1"]=="false"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic.append(mydic1)
                j=j+1
        mydic01 = []                                # stores approved applications
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA1"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic01.append(mydic1)
                j=j+1
        mydic02 = []                                # Stores Rejected applications
        j = 1                                       
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusR1"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic02.append(mydic1)
                j=j+1
    except:
        pass
    return render_template("SupdtDashboard.html", parent_list=mydic,user=username,parent_list01=mydic01,parent_list02=mydic02  )
@app.route('/approversP2/<username>')
def approversP2(username):
    if(not session.get('login_check')):             # checking if user logged-in or not
        return redirect(url_for('login'))
    try:
        todo = db.child("form").get()
        jsondata = todo.val()
        mydic = []                                  # Stores pending applications, 
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA1"]=="true" and i[1]["statusA2"]=="false" and i[1]["statusR2"]=="false"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic.append(mydic1)
                j=j+1
        mydic01 = []
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA2"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic01.append(mydic1)
                j=j+1
        mydic02 = []
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusR2"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic02.append(mydic1)
                j=j+1
    except:
        pass
    return render_template("AR-DR-JR_Dashboard.html", parent_list=mydic,user=username,parent_list01=mydic01,parent_list02=mydic02)
@app.route('/approversP3/<username>')
def approversP3(username):                              # checking if user logged-in or not
    if(not session.get('login_check')):
        return redirect(url_for('login'))
    try:
        todo = db.child("form").get()
        jsondata = todo.val()
        mydic = []                                      # Stores pending applications, 
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA2"]=="true" and i[1]["statusA3"]=="false" and i[1]["statusR3"]=="false"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic.append(mydic1)
                j=j+1
        mydic01 = []
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA3"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic01.append(mydic1)
                j=j+1
        mydic02 = []
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusR3"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic02.append(mydic1)
                j=j+1
    except:
        pass
    return render_template("registrarDashboard.html", parent_list=mydic,user=username,parent_list01=mydic01,parent_list02=mydic02)
@app.route('/approversP4/<username>')
def approversP4(username):
    if(not session.get('login_check')):                     # checking if user logged-in or not
        return redirect(url_for('login'))
    try:
        todo = db.child("form").get()
        jsondata = todo.val()
        mydic = []                                          # Stores pending applications, 
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA3"]=="true" and i[1]["statusA4"]=="false" and i[1]["statusR4"]=="false"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic.append(mydic1)
                j=j+1
        mydic01 = []                                       
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA4"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic01.append(mydic1)
                j=j+1
        mydic02 = []
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusR4"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic02.append(mydic1)
                j=j+1
    except:
        pass
    return render_template("hodDashboard.html", parent_list=mydic,user=username,parent_list01=mydic01,parent_list02=mydic02)
@app.route('/approversA1/<username>')
def approversA1(username):
    if(not session.get('login_check')):
        return redirect(url_for('login'))
    try:
        todo = db.child("form").get()
        jsondata = todo.val()
        mydic = []
        j = 1
        for i in jsondata.items():
            mydic1=[]
            if(i[1]["statusA1"]=="true"):
                mydic1.append(j)
                mydic1.append(i[0])
                mydic1.append(i[1]["date"])
                mydic.append(mydic1)
                j=j+1
    except:
        pass
    return render_template("SupdtDashboard.html", parent_list=mydic,user=username)

@app.route('/approver1Preview/<appId>')                     # used to View the application for supdt, this provides facility to reject or approve by level one
def approver1applicationReview(appId):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    todo = db.child("form").child(appId).get()
    dict1=todo.val()
    dict1["appId"]=appId
    return render_template("approver1Preview.html", dict_item=dict1)
@app.route('/approver2Preview/<appId>')                     # used to View the application for AR/DR/JR, 
def approver2applicationReview(appId):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    todo = db.child("form").child(appId).get()
    dict1=todo.val()
    dict1["appId"]=appId
    return render_template("approver2Preview.html", dict_item=dict1)
@app.route('/approver3Preview/<appId>')
def approver3applicationReview(appId):                      # used to View the application for Registrar
    if(not session['loggedin']):
        return redirect(url_for('login'))
    todo = db.child("form").child(appId).get()
    dict1=todo.val()
    dict1["appId"]=appId
    return render_template("approver3Preview.html", dict_item=dict1)
@app.route('/approver4Preview/<appId>')                     # used to View the application for HOD
def approver4applicationReview(appId):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    todo = db.child("form").child(appId).get()
    dict1=todo.val()
    dict1["appId"]=appId
    return render_template("approver4Preview.html", dict_item=dict1)

@app.route('/approver1Action/<appId>',methods=['GET', 'POST'])      # Takes action of applications by Supdt such as approving or rejecting the application
def approver1Action(appId):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    if (request.form["submit"]=="approved"):
        db.child("form").child(appId).update({"statusA1": "true"})
    else:
        db.child("form").child(appId).update({"statusR1": "true"})
    return redirect(url_for('approversP1',username=session['username']))
@app.route('/approver2Action/<appId>',methods=['GET', 'POST'])          # Takes action of applications by AR/DR/JR such as approving or rejecting the application
def approver2Action(appId):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    if (request.form["submit"]=="approved"):
        db.child("form").child(appId).update({"statusA2": "true"})
    else:
        db.child("form").child(appId).update({"statusR2": "true"})
    return redirect(url_for('approversP2',username=session['username']))
@app.route('/approver3Action/<appId>',methods=['GET', 'POST'])
def approver3Action(appId):                                         # Takes action of applications by Registrar such as approving or rejecting the application
    if(not session['loggedin']):
        return redirect(url_for('login'))
    if (request.form["submit"]=="approved"):
        db.child("form").child(appId).update({"statusA3": "true"})
    else:
        db.child("form").child(appId).update({"statusR3": "true"})
    return redirect(url_for('approversP3',username=session['username']))
@app.route('/approver4Action/<appId>',methods=['GET', 'POST'])      # Takes action of applications by HOD such as approving or rejecting the application
def approver4Action(appId):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    if (request.form["submit"]=="approved"):
        db.child("form").child(appId).update({"statusA4": "true"})
    else:
        db.child("form").child(appId).update({"statusR4": "true"})
    return redirect(url_for('approversP4',username=session['username']))

@app.route('/preview/<appId>')                                      # Allows the indenter to view the application
def applicationReview(appId):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    todo = db.child("form").child(appId).get()
    return render_template("previewin.html", dict_item=todo.val())

@app.route('/search/<int:id>',methods=['POST','GET'])               # Provides facility to search the application by application id
def applicationSearch(id):
    if(not session['loggedin']):
        return redirect(url_for('login'))
    appId=request.form['search']
    if(id==0):
        todo = db.child("form").child(appId).get()
        return render_template("previewin.html", dict_item=todo.val())
    elif(id==1):
            todo = db.child("form").child(appId).get()
            dict1=todo.val()
            dict1["appId"]=appId
            return render_template("approver1Preview.html", dict_item=dict1)
    elif(id==2):
            todo = db.child("form").child(appId).get()
            dict1=todo.val()
            dict1["appId"]=appId
            return render_template("approver2Preview.html", dict_item=dict1)
    elif(id==3):
            todo = db.child("form").child(appId).get()
            dict1=todo.val()
            dict1["appId"]=appId
            return render_template("approver3Preview.html", dict_item=dict1)
    elif(id==4):
            todo = db.child("form").child(appId).get()
            dict1=todo.val()
            dict1["appId"]=appId
            return render_template("approver4Preview.html", dict_item=dict1)
    else:
        pass    
@app.route('/delete/<appId>')
def applicationDelete(appId):                                   # Allows indenter to drop the application
    if(not session['loggedin']):
        return redirect(url_for('login'))
    todo = db.child("form").child(appId).remove()
    return redirect(url_for('myapplications',username=session['username']))

@app.route('/')
@app.route('/login', methods=['GET', 'POST'])                   # This function manages the Authentication part
def login(): 
    msg = ''
    if request.method == 'POST' and 'username' in request.form and 'password' in request.form:
        username = request.form['username']
        password = request.form['password']

        fusername=sheet_instance.find(username)
        fpass=sheet_instance.find(password)
        z=[]
        try:
            print(fusername,fpass)
            loc='E'+str(fpass.row)
            z.append(int(sheet_instance.acell(loc).value))
            print(fpass.row,fpass.col + 1,z)
            if fusername.row==fpass.row and fusername.col!=fpass.col:
                # print(x[0], y[0], z[0])
                session['loggedin'] = True
                session['username'] =fusername.value
                session['login_check']=True
                msg = 'Logged in successfully !'
                if (z[0]==0):
                    return redirect(url_for('myapplications',username=session['username']))
                elif (z[0]==1):
                    return redirect(url_for('approversP1',username=session['username']))
                elif (z[0]==2):
                    return redirect(url_for('approversP2',username=session['username']))
                elif (z[0]==3):
                    return redirect(url_for('approversP3',username=session['username']))                
                elif (z[0]==4):
                    return redirect(url_for('approversP4',username=session['username']))
            else:
                msg = 'Incorrect username / password !'
        except:    
            msg = 'Incorrect username / password !'
    return render_template('login.html',msg=msg)


@app.route('/form')
def form():                                                 # This function manages the to apply new application
    if(not session['loggedin']):
        return redirect(url_for('login'))
    return render_template('form.html')

@app.route('/success/<dic>')
def success(dic):                                           # Manages to view the form
    if(not session['loggedin']):
        return redirect(url_for('login'))
    return render_template('previewin.html',dict_item=session)

@app.route('/logout')
def logout():                                               # 
    session.pop('loggedin', None)
    session.pop('id', None)
    session.pop('username', None)
    session['login_check']=None
    return redirect(url_for('login'))

@app.route('/profile/<username>')
def profile(username):
    fusername=sheet_instance.find(username)
    dic={}
    dic['username']=sheet_instance.acell('B'+str(fusername.row)).value
    dic['id']=sheet_instance.acell('A'+str(fusername.row)).value
    dic['password']=sheet_instance.acell('C'+str(fusername.row)).value
    dic['email']=sheet_instance.acell('D'+str(fusername.row)).value
    L=int(sheet_instance.acell('E'+str(fusername.row)).value)
    if(L==0):
        dic['level']='INDENTER'
        return render_template('profile.html',dic=dic)
    elif(L==1):
        dic['level']='SUPREDENT'
    elif(L==2):
        dic['level']='AR/DR/JR'
    elif(L==3):
        dic['level']='REGISTRAR'
    else:
        dic['level']='HOD'


@app.route('/register', methods=['GET', 'POST'])                 # Manages the registration of new user page
def register():
    msg = ''
    if request.method == 'POST' and 'username' in request.form and 'password' in request.form and 'email' in request.form:
        id=df.shape[0]
        username = request.form['username']
        password = request.form['password']
        email = request.form['email']
        userlevel=request.form['Category']
        fusername=sheet_instance.find(username)
        femail=sheet_instance.find(email)
        # x = df.loc[df['username']==username]['username'].values
        # y = df.loc[df['username']==password]['password'].values
        # z = df.loc[df['username']==password]['level'].values
        r1 = re.compile(r"@iitrpr.ac.in$")
        invalidx = r1.search(email)
        if fusername:
            msg = 'username already exists !'
        elif femail:
            if femail.value:
                msg= 'email already exists !'
        elif username==password:
            msg= 'try another password'
        elif not re.match(r'[^@]+@[^@]+\.[^@]+', email):
            msg = 'Invalid email address !'
        elif not invalidx:
            msg = 'Invalid email Domain !'
        elif not re.match(r'[A-Za-z0-9]+', username):
            msg = 'Username must contain only characters and numbers !'
        elif not username or not password or not email:
            msg = 'Please fill out the form !'
        else:
            sheet_instance.append_row(values=[id,username,password,email,int(userlevel)])
            msg = 'You have successfully registered !'
            return redirect(url_for('login'))
    elif request.method == 'POST':
        msg = 'Please fill out the form !'
    return render_template('register.html', msg=msg)

@app.route('/forgotpassword',methods=['GET','POST'])
def forgotpassword():                                   # Function manages the forget password page
    return render_template('forgotpassword.html')



@app.route('/getotp', methods=['POST','GET'])           # Sends OTP to users email to reset the password
def getotp():                                            
    if request.method=='POST':
        reciver_email=request.form['email']
        session['reset_pass_email']=reciver_email
        otp=random.randint(100000,999999)
        session['otp']=str(otp)
        fromaddr = "add email here"
        password="add pass word here"
        toaddr = reciver_email
        # instance of MIMEMultipart
        msg = MIMEMultipart()
        # storing the senders email address
        msg['From'] = fromaddr
        # storing the receivers email address
        msg['To'] = toaddr
        # storing the subject
        msg['Subject'] = "Reset password"
        # string to store the body of the mail
        body = "Hello enter " +str(otp)+" one time password."
        # attach the body with the msg instance
        msg.attach(MIMEText(body, 'plain'))
        # creates SMTP session
        s = smtplib.SMTP('smtp.gmail.com', 587)
        # start TLS for security
        s.starttls()
        # Authentication
        s.login(fromaddr, password)
        # Converts the Multipart msg into a string
        text = msg.as_string()
        # sending the mail
        s.sendmail(fromaddr, toaddr, text)
        # terminating the session
        s.quit()
        return render_template('enterotp.html')
    return redirect(url_for('forgotpassword'))

@app.route('/resetpassword',methods=['GET','POST'])         # Check whether the OTP entered by the user is correct or not, accordingly proceed
def resetpassword():
    if request.method=='POST' and request.form['otp']:
        print(session['otp'])
        print(request.form['otp'])
        if session['otp']==request.form['otp']:
            return render_template('resetpassword.html')
        else:
            msg="Invalid OTP"
            return render_template('enterotp.html',msg=msg)
    return redirect(url_for('login'))


@app.route('/setpassword',methods=['POST','GET'])               # Confirms whether password and confirmation password is same or not, accordingly changes the password or says Invalid if needed
def setpassword():
    if request.method=='POST' and request.form['password']:
        if request.form['password']==request.form['cnfmpassword']:
            z=session['reset_pass_email']
            cell=sheet_instance.find(z)
            print(cell.value,cell.row,cell.col)
            sheet_instance.update_cell(cell.row, cell.col-1,request.form['password'] )
            return redirect(url_for('setpassword'))
        else:
            msg= "enter same password"
            return render_template('resetpassword.html',msg=msg)
    msg='Password has been updated successfully'
    return render_template('setpassword.html',msg=msg)





@app.route('/action/<username>', methods=['GET', 'POST'])        # It pushes the submitted forms into firebase database
def basic(username):
    if(not session.get('login_check')):
        return redirect(url_for('login'))
    if request.method == 'POST':
        if request.form['submit'] == 'add':
            dic = {}
            dic["user"] = username
            dic["Indenter"] = request.form['Indenter']
            dic["Budget"] = request.form['Budget']
            dic["Cost"] = request.form['Cost']
            dic["Item"] = request.form['Item']
            dic["Category"] = request.form['Category']
            dic["Budgetary_Approval"] = request.form['Budgetary_Approval']
            dic["is_space_ready"] = request.form['is_space_ready']
            dic["is_for_research"] = request.form['is_for_research']
            dic["GeM_Purchase"] = request.form['GeM_Purchase']
            dic["Mode_of_Enquiry"] = request.form['Mode_of_Enquiry']
            dic["no_of_quotations"] = request.form['no_of_quotations']
            today = date.today()
            d1 = today.strftime("%d/%m/%Y")
            dic["date"] = d1
            dic["item_recom"] = request.form['item_recom']
            dic["quotation_no"] = request.form['quotation_no']
            dic["mode_of_payment"] = request.form['mode_of_payment']
            dic["delivery_period"] = request.form['delivery_period']
            dic["statusA1"]="false"
            dic["statusA2"]="false"
            dic["statusA3"]="false"
            dic["statusA4"]="false"
            dic["statusR1"]="false"
            dic["statusR2"]="false"
            dic["statusR3"]="false"
            dic["statusR4"]="false"
            dic["input0name0"] = request.form['input0name0']
            dic["input1name0"] = request.form['input1name0']
            dic["input2name0"] = request.form['input2name0']
            dic["input3name0"] = request.form['input3name0']
            dic["gate"]=request.form['gate']
            dic["tax"] = request.form['tax']
            dic["total"] = request.form['total']
            todo = db.child("form").get()
            print(todo)
            session.update(dic)
            db.child("form").push(dic)
            return redirect(url_for('basic', username = dic['user']))
        elif request.form['submit'] == 'delete':
            db.child("form").remove()
        return redirect(url_for('login'))
    return render_template('previewin.html', dict_item=session)



if __name__ == '__main__':                          # This is a Driver code. 
    
    app.run(debug=True)


## Overview

### The Challenge

Indenter should be able to:

- Can Create form or apply new application
- Can Drop his application
- Can View Status of his application
- Can search the application by application Id
- See hover states for interactive elements
- Can View the application in both grid and List Layout
- Can reset his password if needed

Approvers should be able to:

- Can View all the applications requested by indenters for his approval
- Can Approve or Reject the application
- Can View separately approved, rejected, and Pending applications
- Can search the application by application Id
- See hover states for interactive elements
- Can View the application in both grid and List Layout
- Can reset his password if needed

### Links

- Live Site URL: [https://sahdev1115app.herokuapp.com/login](https://sahdev1115app.herokuapp.com/login)

## Process

### Built with

- Flask
- HTML, CSS Grid and bootstrap
- Google Auth (Google Sheets API)
- Javascript
- Firebase
- Heroku

### Deployment

##### Steps
###### First Time
 - git init
 - git add --all
 - git commit -m "Initial Commit"
 - heroku login
 - heroku create appname
 - git puch heroku master

###### Save Changes
 - git init
 - git add .
 - git commit -m "Initial Commit"
 - git puch heroku main

### requrements
 - Install below packeges in a virtual environment
 - activate the environment
 - Compile app.py : python3 app.py

cachelib==0.6.0
cachetools==5.0.0
certifi==2021.10.8
cffi==1.15.0
charset-normalizer==2.0.12
click==8.1.3
cryptography==37.0.2
Deprecated==1.2.13
Flask==2.1.2
Flask-MySQLdb==1.0.1
Flask-Session==0.4.0
gcloud==0.18.3
google-auth==2.6.6
google-auth-oauthlib==0.5.1
googleapis-common-protos==1.56.0
gspread==5.3.2
gunicorn==20.1.0
httplib2==0.20.4
idna==3.3
importlib-metadata==4.11.3
itsdangerous==2.1.2
Jinja2==3.1.2
jwcrypto==1.2
MarkupSafe==2.1.1
mysql-connector==2.2.9
mysqlclient==2.1.0
numpy==1.22.3
oauth2client==4.1.3
oauthlib==3.2.0
pandas==1.4.2
protobuf==4.21.0rc1
pyasn1==0.4.8
pyasn1-modules==0.2.8
pycparser==2.21
pycryptodome==3.14.1
PyMySQL==1.0.2
pyparsing==3.0.9
Pyrebase4==4.5.0
python-dateutil==2.8.2
python-jwt==3.3.2
pytz==2022.1
requests==2.27.1
requests-oauthlib==1.3.1
requests-toolbelt==0.9.1
rsa==4.8
six==1.16.0
urllib3==1.26.9
Werkzeug==2.1.2
wrapt==1.14.1
zipp==3.8.0
