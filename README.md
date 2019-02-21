using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Text;
using System.Xml;

namespace DocumentVisualizer
{
    public static class DocumentGetSet
    {

        static DocumentGetSet()
        {
            DocInfo = new List<Tuple<string,string>>();
        }

        public static List<Tuple<string, string>> DocInfo { get; set; }

        public static void GetDoc()
        {
            DocInfo.Clear();
            var xmlString = "";
            WebRequest request = WebRequest.Create("http://elasticsearch.prj.elewise.com:8080/docservlet/DocGenerator");
            WebResponse response = request.GetResponse();
            using (Stream stream = response.GetResponseStream())
            {
                using (StreamReader reader = new StreamReader(stream))
                {
                    string line = "";
                    while ((line = reader.ReadLine()) != null)
                    {
                        xmlString += line;
                    }
                }
            }
            response.Close();


            XmlDocument xDoc = new XmlDocument();
            xDoc.LoadXml(xmlString);
            //xDoc.Load(xmlString);
            // получим корневой элемент
            XmlElement xRoot = xDoc.DocumentElement;
            // обход всех узлов в корневом элементе
            foreach (XmlNode xnode in xRoot)
            {
                GetTextFromNode(xnode);

            }
        }


        public static void GetTextFromNode(XmlNode node)
        {
            if (node.Attributes != null)
            {
                foreach (var attr in node.Attributes.OfType<XmlAttribute>())
                {
                    DocInfo.Add(new Tuple<string, string>(attr.Name, attr.Value));
                }
            }
            // обходим все дочерние узлы элемента user
            foreach (XmlNode childnode in node.ChildNodes)
            {
                GetTextFromNode(childnode);
            }
        }
    }
}
