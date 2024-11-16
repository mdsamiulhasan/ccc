
public ActionResult Index()

        {
            var student = db.students.Include(a=>a.addmitions.Select(c=>c.course)).OrderBy(s=>s.sid).ToList();
            return View(student);
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
            if (ModelState.IsValid) {
                var st = new student()
                {
                    sname = student.sname,
                    age = student.age,
                    ismorning = student.ismorning,
                    mark=student.mark,                    
                };
                HttpPostedFileBase file = student.pictureFile;
                if (file != null) {
                    string filename = Path.Combine("/Images/", DateTime.Now.Ticks.ToString()+ Path.GetExtension(file.FileName));
                    file.SaveAs(Server.MapPath(filename));
                    st.picture = filename;
                }
                foreach (var c in cid) {
                    addmition a = new addmition()
                    {
                        student = st,
                        sname = student.sname,
                        sid = student.sid,
                        cid = c,
                    };
                    db.addmitions.Add(a);
                  
                }
                db.SaveChanges();
                return RedirectToAction("Index");
            }
            return View(student);
        }
        // Partial
        public ActionResult AddCourse(int? id)
        {
            ViewBag.course = new SelectList(db.courses.ToList(), "cid", "cname", id ?? 0);
            return PartialView("_addCourse");
        }

        //edit
        public ActionResult Edit(int ? id)
        {
            var student = db.students.Find(id);
            return View(student);
        }
        [HttpPost]
        public ActionResult Edit(student student, int[] cid)
        {
            if (ModelState.IsValid)
            {              
                var st = db.students.Find(student.sid);
                    st.sname = student.sname;
                    st.age = student.age;
                    st.ismorning = student.ismorning;
                    st.mark = student.mark;
               
                HttpPostedFileBase file = student.pictureFile;
                if (file != null)
                {
                    string filename = Path.Combine("/Images/", DateTime.Now.Ticks.ToString() + Path.GetExtension(file.FileName));
                    file.SaveAs(Server.MapPath(filename));
                    st.picture = filename;
                }
                else {
                    st.picture = st.picture;
                }          
                var a = db.addmitions.Where(c => c.sid == student.sid);
                db.addmitions.RemoveRange(a);
                foreach (var c in cid)
                {
                    addmition ad = new addmition()
                    {
                        student = st,
                        sname = student.sname,
                        sid = student.sid,
                        cid = c,
                    };
                    db.addmitions.Add(ad);
                }
                db.SaveChanges();
                return RedirectToAction("Index");
            }
            return View(student);
        }
        //delete
        public ActionResult Delete(int? id)
        {
            var student = db.students.Find(id);
            var a = db.addmitions.Where(c => c.sid == student.sid);
            db.addmitions.RemoveRange(a);
            db.students.Remove(student);
            db.SaveChanges();
            return RedirectToAction("Index");
        }
