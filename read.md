 public ActionResult Index()
 {
     var students = db.students.Include(a => a.admissions.Select(c => c.course)).OrderBy(s => s.sid).ToList();
     return View(students);
 }
 //create
 public ActionResult Create()
 {
     return View();
 }
 //post
 [HttpPost]
 public ActionResult Create(student student, int[] cid)
 {
     var st = new student()
     {
         sid = student.sid,
         sname = student.sname,
         age = student.age,
         mark = student.mark,
         ismorning = student.ismorning,

     };

     HttpPostedFileBase file = student.pictureFile;
     if (file != null)
     {
         string filename = Path.Combine("/images/", DateTime.Now.Ticks.ToString() + Path.GetExtension(file.FileName));
         file.SaveAs(Server.MapPath(filename));
         st.picture = filename;
     }
     else
     {
         st.picture = st.picture;
     }
     var a = db.admissions.Where(c => c.cid == student.sid);
     db.admissions.RemoveRange(a);
     foreach (var c in cid)
     {
         admission ad = new admission()
         {
             student = st,
             sname = student.sname,
             sid = student.sid,
             cid = c,
         };
         db.admissions.Add(ad);

     }
     db.SaveChanges();
     return RedirectToAction("Index");
 }
 //partial
 public ActionResult Addmore(int ?id)
 {
     ViewBag.course = new SelectList(db.courses.ToList(),"cid","cname", id??0);
  return  PartialView("_addmore");
 }
 //edit
 public ActionResult Edit(int? id)
 {
     var student = db.students.Find(id);
     var a = db.admissions.Where(c => c.sid == student.sid).ToList();
     student.admissions = a;
     return View(student);
 }
 //edit post
 [HttpPost]
 [ValidateAntiForgeryToken]
 public ActionResult Edit(student student, HttpPostedFileBase pictureFile)
 {
     if (ModelState.IsValid)
     {

         if (pictureFile != null)
         {
             var filePath = Path.Combine(Server.MapPath("~/images"), pictureFile.FileName);
             pictureFile.SaveAs(filePath);
             student.picture = "/Images/" + pictureFile.FileName;
         }
         db.Entry(student).State = EntityState.Modified;
         db.SaveChanges();
         return RedirectToAction("Index");
     }
     return View(student);
 }
 //delete
 public ActionResult Delete(int?id)
 {
     var student = db.students.Find(id);
     var a = db.admissions.Where(c => c.sid == student.sid);
     db.admissions.RemoveRange(a);
     db.students.Remove(student);
     db.SaveChanges();
     return RedirectToAction("Index");
 }
