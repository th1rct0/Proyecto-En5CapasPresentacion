# Proyecto-En5CapasPresentacion
Modulo de presentación terminado, consulta desde el Id y presentación de la información en el formulario

####FORMULARIO


using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

using System.Data.SqlClient;
using Entidades;


namespace VT_Ca
{
    public partial class Form1 : Form
    {
        P_Customer oCliente = new P_Customer();// Instancia de la clase de la capa de presentación
        E_Customer oCliente_Ent = new E_Customer();//instancia de la clase entidad
        public Form1()
        {
            InitializeComponent();
        }
        private void IniDG()  // COn INidg se llena el datagridview
        {

            try
            {

                DGCustomer.DataSource = oCliente.GetAll().Tables[0];
                DGCustomer.Refresh();

            }
            catch (Exception Ex)
            {
                MessageBox.Show("Formulario--->   " +Ex.Message);
            }
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            IniDG();
        }

        private void txtCustomerId_KeyPress(object sender, KeyPressEventArgs e)
        {//cuando tecleas enter en el textbox
            if (e.KeyChar == (char)Keys.Return)
            {   int CusId= Convert.ToInt32(txtCustomerId.Text);

                try
                {
                    oCliente_Ent = oCliente.GetOne(CusId);
                    chkNamSTyle.Checked = oCliente_Ent.NameStyle;
                    txtTitle.Text = oCliente_Ent.Title;
                    txtFirstName.Text = oCliente_Ent.FirstName;
                    txtMiddleName.Text = oCliente_Ent.MiddleName;
                    txtLastName.Text = oCliente_Ent.LastName;
                    txtSuffiix.Text = oCliente_Ent.Suffix;
                    txtCompanyName.Text = oCliente_Ent.CompanyName;
                   textBoxtxtSalesPerson.Text = oCliente_Ent.SalesPerson;
                    txtEmailAddress.Text = oCliente_Ent.EmailAddress;
                    txtPhone.Text = oCliente_Ent.Phone;
                    txtPasswordhash.Text = oCliente_Ent.PasswordHash;
                    txtPasswordSalt.Text = oCliente_Ent.PasswordSalt;


                }
                catch(Exception Ex)
                {
                    MessageBox.Show("TextBox--->   " + Ex.Message);
                }
                
            }
        }

        private void btnGrabar_Click(object sender, EventArgs e)
        {
            /*Aqui se graba la informacion en la base de datos
             
             
             */

            oCliente_Ent.CustomerId = Convert.ToInt32((txtCustomerId.Text == string.Empty) ? null : txtCustomerId.Text);
            //si el valor del txtCustomerId es cadena vacia asigna null, de lo contrario asigna el valor del textbox
            oCliente_Ent.NameStyle = chkNamSTyle.Checked;
            oCliente_Ent.Title = txtTitle.Text;
            oCliente_Ent.FirstName = txtFirstName.Text;
            oCliente_Ent.MiddleName = txtMiddleName.Text;
            oCliente_Ent.LastName = txtLastName.Text;
            oCliente_Ent.Suffix = txtSuffiix.Text;
            oCliente_Ent.CompanyName = txtCompanyName.Text;
            oCliente_Ent.SalesPerson = textBoxtxtSalesPerson.Text;
            oCliente_Ent.EmailAddress = txtEmailAddress.Text;
            oCliente_Ent.Phone = txtPhone.Text;
            oCliente_Ent.PasswordHash = txtPasswordhash.Text;
            oCliente_Ent.PasswordSalt = txtPasswordSalt.Text;

            int vCustid = -1;//Valor del CustId, si es -1 indica que no se grabo correctamente
            try {

                /* */
                 vCustid = oCliente.Insert(oCliente_Ent);
               
                
            }
            catch {
                vCustid = -1;//Si hubo error se pone -1
                MessageBox.Show("Error al Grabar el Registro");

            }

            if (vCustid != -1)
            {
                /*Si se grabo bien vCust tiene un valor diferente de -1 */
                txtCustomerId.Text = Convert.ToString(vCustid);
             oCliente_Ent.CustomerId = vCustid;
               
                MessageBox.Show("Registro Grabado Correctamente. CustId: " + vCustid.ToString());



            }
            else
            {
                MessageBox.Show("Error al Grabar el Registro");
            }

            IniDG();//Refresca el datagridview
            //Inicializar DataGridView y enviar datos 
            txtCustomerId_KeyPress(sender, new KeyPressEventArgs((char)Keys.Return));//refresca los datos del formulario
        }
    }
}

################P_Cus
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using System.Data;
using System.Data.SqlClient;
using Entidades;
using Negocio;



namespace VT_Ca
{
    public class P_Customer
    {
        N_Customer N_Cus = new N_Customer();  // Instance of Business Logic Layer for Customer
        public DataSet GetAll()
        {
            return N_Cus.GetAll(); // Method to retrieve all customers from the Business Logic Layer
        }
        public E_Customer GetOne(int Customer_Id)
        {
            return N_Cus.GetOne(Customer_Id); // 

        }
        public int Insert(E_Customer oCustomer)
        {
            return N_Cus.Insert(oCustomer); // Method to insert a new customer into the database via the Data Access Layer
        }

    }
}


#######D_Customer
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using System.Data;
using System.Data.SqlClient;
using Entidades;
using System.Runtime.Remoting.Messaging;

namespace Datos
{
    public class D_Customer : CAD //tener acceso a la clase DEBSAccesoDatos para conectarnos a la base de datos
    {
        public DataSet GetAll()// Method to retrieve all customers
        {
            DataSet dataSet = new DataSet();
            DataSet DS = dataSet;

            try
            {


                SqlCommand cmd = CrearComando("Customer_Get");// Create a SqlCommand to execute the stored procedure "Customer_Get"
                DS = GetDS(cmd, "Customer_Get");// Execute the command and get the DataSet

            }
            catch (SqlException Ex)// Catch any exceptions that occur during the process

            {
                throw new Exception("Error Obteniendo todos los registros en D_CUSTOMER   " + Ex.Message, Ex);

            }




            return DS;
        }

        public E_Customer GetOne(int CustomerId)
        {
            E_Customer vRes = new E_Customer();

            SqlCommand cmd = new SqlCommand();
            try
            {
                cmd = CrearComando("Customer_Get");
                cmd.Parameters.AddWithValue("@CustomerId", CustomerId);

                AbrirConexion();
                SqlDataReader consulta = Ejecuta_Consulta(cmd);
                if (consulta.Read())
                {
                    if (consulta.HasRows)
                    {
                        vRes.CustomerId = (int)consulta["CustomerId"];
                        vRes.NameStyle = (Boolean)consulta["NameStyle"];/*//Boolean no puede ser null. con B mayuscula Más común en código que usa reflexión o interoperabilidad
                                                                            Tiene métodos como Parse(), ToString(), etc.Pertenece a System*/

                        vRes.Title = (string)consulta["Title"]; //puede ser null
                        vRes.FirstName = (string)consulta["FirstName"];
                        vRes.MiddleName = Convert.ToString(consulta["MiddleName"]);
                        vRes.LastName = (string)consulta["LastName"];
                        vRes.Suffix = Convert.ToString(consulta["Suffix"]);
                        vRes.CompanyName = Convert.ToString(consulta["CompanyName"]);
                        vRes.SalesPerson = Convert.ToString(consulta["SalesPerson"]);
                        vRes.EmailAddress = Convert.ToString(consulta["EmailAddress"]);
                        vRes.Phone = Convert.ToString(consulta["Phone"]);
                        vRes.PasswordHash = (string)consulta["PasswordHash"];
                        vRes.PasswordSalt = (string)consulta["PasswordSalt"];
                        vRes.ModifiedDate = (DateTime)consulta["ModifiedDate"];
                    }
                }
                //   return vRes;
                consulta.Close();
                consulta.Dispose();

            }
            catch (SqlException Ex)
            {
                throw new Exception("Error Obteniendo un registro en D_CUSTOMER   " + Ex.Message, Ex);
            }
            finally { 
                cmd.Dispose();
                CerrarConexion();
            }
            return vRes;
        }
        public int Insert(E_Customer oCustomer)
        {
            int vRes = -1;
            SqlCommand cmd = new SqlCommand();
            try
            {
                cmd = CrearComando("Customer_set");
                cmd.Parameters.AddWithValue("@CustomerId", oCustomer.CustomerId);
                cmd.Parameters["@CustomerId"].Direction = ParameterDirection.Input;

                cmd.Parameters.AddWithValue("@NameStyle", oCustomer.NameStyle);
                cmd.Parameters.AddWithValue("@Title", oCustomer.Title);
               
                cmd.Parameters.AddWithValue("@FirstName", oCustomer.FirstName);
                cmd.Parameters.AddWithValue("@MiddleName", oCustomer.MiddleName);
                cmd.Parameters.AddWithValue("@LastName", oCustomer.LastName);
                cmd.Parameters.AddWithValue("@Suffix", oCustomer.Suffix);
                cmd.Parameters.AddWithValue("@CompanyName", oCustomer.CompanyName);
                cmd.Parameters.AddWithValue("@SalesPerson", oCustomer.SalesPerson);
                cmd.Parameters.AddWithValue("@EmailAddress", oCustomer.EmailAddress);
                cmd.Parameters.AddWithValue("@Phone", oCustomer.Phone);
                cmd.Parameters.AddWithValue("@PasswordHash", oCustomer.PasswordHash);
                cmd.Parameters.AddWithValue("@PasswordSalt", oCustomer.PasswordSalt);
               // cmd.Parameters.AddWithValue("@ModifiedDate", oCustomer.ModifiedDate);
                AbrirConexion();
                vRes = Ejecuta_Accion(ref cmd);
                vRes = Convert.ToInt32(cmd.Parameters["@CustomerId"].Value);

            }
            catch (Exception Ex)
            {
                throw new Exception("Error Insertando un registro en D_CUSTOMER   " + Ex.Message, Ex);
            }
            finally
            {
                cmd.Dispose();//Limpiar recursos
                CerrarConexion();
            }
            return vRes;
        }
    }
}


#########N_Customer
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using System.Data.SqlClient;
using System.Data;

using Entidades; //tener acceso a la clase Entidades
using Datos;  //tener acceso a la clase Datos

namespace Negocio
{
    public class N_Customer
    {
        D_Customer D_Cus = new D_Customer(); // Instance of Data Access Layer for Customer

        public DataSet GetAll()
        {
            return D_Cus.GetAll(); // Method to retrieve all customers from the Data Access Layer

        }

        public E_Customer GetOne(int Customer_Id)
        {
            return D_Cus.GetOne(Customer_Id); // Method to retrieve a single customer by ID from the Data Access Layer /Recivora un dato una fila de la tabla
        }
        public int Insert(E_Customer oCustomer) 
        { 
         return D_Cus.Insert(oCustomer); // Method to insert a new customer into the database via the Data Access Layer
        }

    }

}


Se tienen que modificar estos archivos y agregar un boton para Grabar el dato.
Tambien un metodo en el Cuadro de texto para buscar al presionar enter.
