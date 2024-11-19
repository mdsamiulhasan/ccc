
 public ActionResult Index()
 {
     var students = db.students.Include(c => c.admissions.Select(b => b.course)).OrderBy(x => x.sid).ToList();
     return View(students);
 }
 //create
 public ActionResult Create()
 {
    
     return View();
 }
      

 [HttpPost]
 public ActionResult Create(student student, int[] cid)
 {
     var st = new student
     {
         sid = student.sid,
         sname = student.sname,
         age = student.age,
         mark = student.mark,
         picture = SaveImage(student.pictureFile)
     };        
     db.admissions.RemoveRange(db.admissions.Where(c => c.sid == student.sid));  
     foreach (var c in cid)
     {
         db.admissions.Add(new admission { student = st, sid = student.sid, cid = c });
     }

     db.SaveChanges();
     return RedirectToAction("Index");
 }
 private string SaveImage(HttpPostedFileBase file)
 {
     if (file == null) return null;

     var filePath = Path.Combine("/images", DateTime.Now.Ticks + Path.GetExtension(file.FileName));
     file.SaveAs(Server.MapPath(filePath));
     return filePath;
 }


 //partial View
 public ActionResult Addcourse(int? id)
 {
     ViewBag.courses = new SelectList(db.courses.ToList(), "cid", "cname", id ?? 0);
     return PartialView("_addcourse");
 }
 //delete
 public ActionResult Delete(int? id)
 {
     db.admissions.RemoveRange(db.admissions.Where(c => c.sid == id));
     db.students.Remove(db.students.Find(id));
     db.SaveChanges();
     return RedirectToAction("Index");
 }

 //edit
 public ActionResult Edit(int? id)
 {
     var students = db.students.Find(id);
     return View(students);
 }

 [HttpPost]
 public ActionResult Edit(student student, HttpPostedFileBase pictureFile)
 {
     if (ModelState.IsValid)
     {
         if (pictureFile != null)
         {              
             var fileName = DateTime.Now.Ticks.ToString() + Path.GetExtension(pictureFile.FileName);
             var filePath = Path.Combine(Server.MapPath("~/images"), fileName);         
             pictureFile.SaveAs(filePath);
             student.picture = "/images/" + fileName;
         }
         db.Entry(student).State = EntityState.Modified;
         db.SaveChanges();
         return RedirectToAction("Index");
     }
     return View(student);
 }
