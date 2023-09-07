# FormFlask
En este código de Python estoy utilizando flask para la realización de una aplicación de register y login. Lo hace la aplicación es crear una base de datos llamada "users" con los siguientes campos (user, email, password y password confirmation), esa base de datos será completada cuando el usuario complete el formulario de registro que pide lo siguiente (usuario, email, contraseña y confirmación de contraseña).
En la parte de login, el usuario tiene que completar el formulario de logueo que pide lo siguiente (usuario, email y contraseña), se debe verificar con la base de datos (anteriormente creada) los datos ingresados por el usuario. Si, alguna de las tres cosas pedidas no coinciden con la base de datos aparece un mensaje "Los datos ingresados son incorrectos, vuelve a intentarlo" y en le caso de que los datos sean correctos "Bienvenido + usuario".
Este es el código:
from flask import Flask, render_template, request, redirect, url_for, session
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)

@app.route('/')
def index():
    @app.route('/registro', methods=['GET', 'POST'])
    def registro():
        if request.method == 'POST':
            username = request.form['username']
        email = request.form['email']
        password = request.form['password']
        password_repeat = request.form['password_repeat']

        if password == password_repeat:
            hashed_password = generate_password_hash(password, method='sha256')
            new_user = User(username=username, email=email, password=hashed_password)
            db.session.add(new_user)
            db.session.commit()
            return "Registro exitoso. Ahora puedes iniciar sesión."
        else:
            return "Las contraseñas no coinciden. Inténtalo de nuevo."

    return render_template('registro.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        email = request.form['email']
        password = request.form['password']

        user = User.query.filter_by(username=username).first()

        if user and check_password_hash(user.password, password):
            session['username'] = username
            return "Bienvenido, " + username
        else:
            return "Los datos ingresados son incorrectos. Vuelve a intentarlo."

    return render_template('login.html')

app.secret_key = 'tu_clave_secreta'  # Cambia esto a una clave secreta más segura
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
db = SQLAlchemy(app)


class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(120), nullable=False)


if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(debug=True)

